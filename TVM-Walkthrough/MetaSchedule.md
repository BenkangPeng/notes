### Refer

[Introducing TVM Auto-scheduler (a.k.a. Ansor)](https://tvm.apache.org/2021/03/03/intro-auto-scheduler)

[MetaSchedule | TVM Discuss RFC](https://discuss.tvm.apache.org/t/rfc-meta-schedule-autotensorir/10120/17)

### Feature Extraction

FeatureExtractor对一个Scheduled IRModule中每一个BufferStoreNode提取出**164**个特征(浮点计算个数、访存量、计算访存比等等)。详见[per_store_feature.cc at v0.20.0 · apache/tvm](https://github.com/apache/tvm/blob/v0.20.0/src/meta_schedule/feature_extractor/per_store_feature.cc)

有一个测试样例，可以对照着数值看

[test_meta_schedule_feature_extractor_per_store_feature.py at v0.20.0 · apache/tvm Line236](https://github.com/apache/tvm/blob/v0.20.0/tests/python/meta_schedule/test_meta_schedule_feature_extractor_per_store_feature.py#L236)

用特征向量当数据，执行时间为labels，用xgboost来学习程序的特征向量，根据程序的特征向量预测执行时间

> # Explaining the PackSum Class in TVM's XGBoost Cost Model
> 
> Let me walk you through the `PackSum` class, which is a key component in TVM's XGBoost-based cost model for predicting performance of tensor operations. I'll explain it step by step, including the necessary XGBoost background.
> 
> ## XGBoost Background
> 
> Before diving into `PackSum`, let's understand some XGBoost basics:
> 
> 1. **XGBoost**: A powerful gradient boosting library for machine learning, especially effective for tabular data and regression problems.
> 
> 2. **DMatrix**: XGBoost's specialized data structure for storing feature data and labels efficiently.
> 
> 3. **Gradient Boosting**: An ensemble technique that builds trees sequentially, with each tree correcting the errors of previous trees.
> 
> 4. **Objective Functions**(目标函数): Define what the model is trying to optimize (e.g., minimizing prediction error).
> 
> ## The Problem PackSum Solves
> 
> In TVM's meta-schedule, we have a unique challenge:
> 
> 1. Each tensor operation candidate (schedule) produces multiple "blocks"(@benkangpeng: I think it's BufferStoreNode instead of block) of features (one per store operation)
>    
>    一个IRModule(candidate or schedule)，可能有多个BufferStoreNode; 一个BufferStoreNode产生(1,164)的feature vector; 因此一个IRModule会产生(n,164)的feature vector，其中n为BufferStoreNode的个数。
> 
> 2. But we want a single performance prediction for the entire schedule
> 
> 3. The actual performance (runtime) is a property of the whole schedule, not individual blocks
> 
> This is why we need `PackSum`: to aggregate predictions from multiple blocks into a single score.
> 
> ## PackSum Class Explained
> 
> ### Class Definition and Constructor
> 
> ```python
> class PackSum:
>     """The pack-sum format"""
> 
>     dmatrix: "xgb.DMatrix"
>     ids: np.ndarray
> 
>     def __init__(
>         self,
>         xs: List[np.ndarray],
>         ys: Optional[np.ndarray],
>     ):
> ```
> 
> - `dmatrix`: An XGBoost data matrix holding feature vectors from all blocks
> - `ids`: Maps each block back to its parent schedule/sample
> - `xs`: List of feature arrays, one per schedule candidate
> - `ys`: Optional labels (actual runtimes) for training
> 
> ### Constructor Implementation
> 
> ```python
> import xgboost as xgb
> 
> repeats = [x.shape[0] for x in xs]
> xs = np.concatenate(xs, axis=0)
> self.ids = np.concatenate([[i] * repeat for i, repeat in enumerate(repeats)], axis=0)
> ```
> 
> **Example**:
> If you have 3 schedule candidates with 4, 2, and 3 BufferStoreNode respectively:
> 
> - `repeats = [4, 2, 3]`
> - `xs` becomes a single array with 9 rows (4+2+3)
> - `self.ids = [0,0,0,0, 1,1, 2,2,2]` (showing which candidate each block belongs to)
> 
> ```python
> if ys is None:
>     self.dmatrix = xgb.DMatrix(data=xs, label=None)
> else:
>     ys = np.concatenate([[y] * repeat for y, repeat in zip(ys, repeats)], axis=0)
>     self.dmatrix = xgb.DMatrix(data=xs, label=ys)
>     self.dmatrix.set_weight(ys)
> ```
> 
> If labels are provided:
> 
> - Each label is repeated for every block from the same candidate
> - Example: if `ys = [1.5, 2.7, 0.9]` (runtimes for 3 candidates), it becomes `[1.5,1.5,1.5,1.5, 2.7,2.7, 0.9,0.9,0.9]`
> - These are used as both labels and weights in the XGBoost matrix
> 
> ### predict_with_score Method
> 
> ```python
> def predict_with_score(self, pred: np.ndarray) -> np.ndarray:
>     """Predict the labels given the block level prediction scores."""
>     return np.bincount(self.ids, weights=pred)
> ```
> 
> This is the key aggregation function:
> 
> - `pred`: Block-level predictions from the XGBoost model
> - The function sums up predictions for blocks belonging to the same candidate
> - `np.bincount(self.ids, weights=pred)` adds up values in `pred` according to the groups in `self.ids`
> 
> **Example**:
> 
> - If `pred = [0.2, 0.3, 0.15, 0.25, 0.5, 0.4, 0.1, 0.2, 0.3]`
> - And `self.ids = [0,0,0,0, 1,1, 2,2,2]`
> - Result = `[0.2+0.3+0.15+0.25, 0.5+0.4, 0.1+0.2+0.3]` = `[0.9, 0.9, 0.6]`
> 
> ### obj_square_error Method
> 
> ```python
> def obj_square_error(self, ys_pred: np.ndarray) -> Tuple[np.ndarray, np.ndarray]:
>     """Implement square error loss on pack-sum format as
>     a custom objective function for xgboost."""
> 
>     # Making prediction
>     ys_pred = self.predict_with_score(ys_pred)
>     # Propagate prediction to each block
>     ys_pred = ys_pred[self.ids]
>     # The gradient and hessian
>     ys = self.dmatrix.get_label()
>     gradient = ys_pred - ys
>     hessian = np.ones_like(gradient)
>     return gradient * ys, hessian * ys
> ```
> 
> This is a custom objective function for XGBoost that:
> 
> 1. Aggregates block predictions to candidate-level predictions
> 2. Propagates these back to block level (each block gets its candidate's prediction)
> 3. Calculates gradient (prediction - actual) and constant hessian for XGBoost's optimization
> 4. Weights both by the labels (giving more importance to high-runtime samples)
> 
> ### rmse Method
> 
> ```python
> def rmse(self, ys_pred: np.ndarray) -> Tuple[str, float]:
>     """Evaluate RMSE (rooted mean square error) in the pack-sum format"""
> 
>     # Making prediction
>     ys_pred = self.predict_with_score(ys_pred)
>     # Propagate prediction to each block
>     ys_pred = ys_pred[self.ids]
>     # The RMSE
>     ys = self.dmatrix.get_label()
>     square_error = np.square(ys_pred - ys)
>     rmse = np.sqrt(square_error.mean())
>     return "p-rmse", rmse
> ```
> 
> This calculates the root mean squared error between:
> 
> - The aggregated predictions (summed across blocks for each candidate)
> - The actual runtime labels
> - Returns both the metric name "p-rmse" (packed RMSE) and the score
> 
> ### average_peak_score Method
> 
> ```python
> def average_peak_score(
>     self,
>     ys_pred: np.ndarray,
>     n: int,
> ) -> Tuple[str, float]:
>     """Evaluate average-peak-score@N in the pack-sum format"""
> 
>     ys = self.dmatrix.get_label()
>     ys = self.predict_with_score(ys)
>     ys = ys / np.unique(self.ids, return_counts=True)[1]
>     ys_pred = self.predict_with_score(ys_pred)
>     trials = np.argsort(ys_pred)[::-1][:n]
>     trial_scores = ys[trials]
>     curve = max_curve(trial_scores) / np.max(ys)
>     score = np.mean(curve)
>     return f"a-peak@{n}", score
> ```
> 
> This evaluates how well the top-N predictions perform:
> 
> 1. Normalizes true runtimes by the number of blocks in each candidate
> 2. Sorts candidates by predicted performance (best first)
> 3. Selects the top-N candidates
> 4. Calculates a "max curve" (running maximum of performance)
> 5. Returns the average normalized peak score
> 
> ## Putting It All Together: How PackSum Works in Practice
> 
> Here's a complete example of how `PackSum` would be used in TVM's cost model:
> 
> 1. **Feature Extraction**:
>    
>    - 3 schedule candidates with [4, 2, 3] blocks each
>    - Each block produces a feature vector (e.g., length 164)
>    - Features describe memory access patterns, loop structure, etc.
> 
> 2. **Creating PackSum**:
>    
>    - `xs` = List of feature arrays with shapes [(4,164), (2,164), (3,164)]
>    - `ys` = [1.5, 2.7, 0.9] (runtimes in milliseconds)
>    - `PackSum(xs, ys)` creates the packed structure
> 
> 3. **Training XGBoost**:
>    
>    - XGBoost model uses `PackSum.obj_square_error` as objective
>    - Each training iteration uses `PackSum.rmse` to evaluate progress
> 
> 4. **Making Predictions**:
>    
>    - Model predicts for each block individually
>    - `PackSum.predict_with_score` aggregates block predictions to get candidate scores
>    - Candidates with lowest predicted scores are selected for actual execution
> 
> The magic of `PackSum` is that it allows XGBoost to learn from block-level features while predicting schedule-level performance, bridging the gap between the granularity of feature extraction and the performance metric we care about.

`tune`的入口在`task_scheduler.cc` Line 162

```cpp
    Array<tir::Schedule> design_spaces = 
ctx->space_generator.value()->GenerateDesignSpace(ctx->mod.value());
```

`post_order_apply`后续遍历所有`block` 生成`design space`

### Post Order Apply

`post_order_apply.cc` `GenerateDesignSpace`

对`sch`每一个`block`逆序依次应用内置的schedule rules

例如，对于以下的matmul tir module, 它(sch)有两个blocks: `main` and `C`

```cpp
    @tvm.script.ir_module
    class MyTirModule:
        @T.prim_func
        def matmul(A: T.Buffer((128, 128), "float32"), B: T.Buffer((128, 128), "float32"), C: T.Buffer((128, 128), "float32")):  # type: ignore
            T.func_attr({"global_symbol": "main", "tir.noalias": True})
            for i, j, k in T.grid(128, 128, 128):
                with T.block("C"):
                    vi, vj, vk = T.axis.remap("SSR", [i, j, k])
                    with T.init():
                        C[vi, vj] = 0.0
                    C[vi, vj] += A[vi, vk] * B[vk, vj]
```

默认的Schedule rules:

```
[meta_schedule.ApplyCustomRule(0x55555c330f58), 
meta_schedule.InlineConstantScalars(0x55555bfdbd78), 
meta_schedule.AutoInline(0x55555c35c788), 
meta_schedule.AddRFactor(0x55555c35c7f8), 
meta_schedule.MultiLevelTiling(0x55555c1e2108), 
meta_schedule.ParallelizeVectorizeUnroll(0x55555c353908), 
meta_schedule.RandomComputeLocation(0x55555c2a6f28)]
```

* ApplyCustomRule
  
  ```cpp
    // Inherited from ScheduleRuleNode
    Array<tir::Schedule> Apply(const tir::Schedule& sch, const tir::BlockRV& block_rv) final {
      CHECK(this->target_.defined())
          << "ValueError: ApplyCustomRule is not initialized with TuneContext that has a Target.";
      Array<String> keys = this->target_.value()->k
    eys;//获取annotation of custom shedule rules
      if (Optional<String> ann = tir::GetAnn<String>(sch->GetSRef(block_rv), "schedule_rule")) {
        if (ann.value() != "None") {
          for (const String& key : keys) {
            if (const runtime::PackedFunc* custom_schedule_fn =
                    runtime::Registry::Get(GetCustomRuleName(ann.value(), key))) {
              //implement the custom schedule rules
    Array<tir::Schedule> result = ((*custom_schedule_fn)(sch, block_rv));
              return result;
            }
          }
  ```

* AddRFactor

这个rules怎么没效果呢？再测

* MultiLevelTiling
  
  ```cpp
  // Entry of the mega rule; Inherited from ScheduleRuleNode
  Array<Schedule> MultiLevelTilingNode::Apply(const Schedule& sch, const BlockRV& block_rv) {
    if ((filter_fn_ && filter_fn_.value()(sch, sch->GetSRef(block_rv))) ||
        NeedsMultiLevelTiling(sch->state(), sch->GetSRef(block_rv))) {
      sch->Annotate(block_rv, tir::attr::meta_schedule_tiling_structure, structure);
  
      Array<Schedule> results;
      for (auto&& state : ApplySubRules({State(sch, block_r
  v)})) {///Use 3 sub-schedule rules to generate 3 new schedule
        results.push_back(std::move(state->sch));
      }
      return results;
    }
    return {sch};
  }
  ```

* ParallelizeVectorizeUnroll

这个schedule rule只是直接给block加上了并行/向量化/展开的annotation，未检测调度正确性.例如

```python
@I.ir_module
class Module:
    @T.prim_func
    def main(A: T.Buffer((128, 128), "float32"), B: T.Buffer((128, 128), "float32"), C: T.Buffer((128, 128), "float32")):
        T.func_attr({"tir.noalias": T.bool(True)})
        with T.block("root"):
            T.reads()
            T.writes()
            T.block_attr({"meta_schedule.parallel": 16, "meta_schedule.unroll_explicit": 16, "meta_schedule.vectorize": 64})
            for i_0, j_0, i_1, j_1, k_0, i_2, j_2, k_1, i_3, j_3 in T.grid(4, 2, 1, 64, 64, 8, 1, 2, 4, 1):
                with T.block("C"):
```

### Evolutionary Search

`evolutionary_search.cc Line 699`  `GenerateMeasureCandidates`

This function generate some(e.g. 512) candidates(or schedule), while one part is from the database and another is generated by `sampling`.

```cpp
/*! The first part of candidates. */
/*! \note Pick the top K schedule from historical data stored in database */
std::vector<Schedule> measured = PickBestFromDatabase(pop * self->init_measured_ratio);
  TVM_PY_LOG(INFO, self->ctx_->logger)
      << "Picked top " << measured.size() << " candidate(s) from database";

/*! The second part */
std::vector<Schedule> unmeasured = SampleInitPopulation(pop - measured.size());
```

The design space(3 schedules) generated at the stage of `Post Order Apply`, is called `sketch`(purposed in Ansor 20'OSDI). `sketch` indicates the structure of program, including the order of loop, loop fusion and so on.



We can print the `sketch` stored in design space generated at the stage of `post order apply`.

```
[_ = sch.get_block(name=C, func_name=main), 
_ = sch.get_block(name=root, func_name=main), 
sch.annotate(block_or_loop=_, 
ann_key=meta_schedule.tiling_structure, ann_val="SSRSRS"), 
_, _, _ = sch.get_loops(block=_), 
_, _, _, _ = sch.sample_perfect_tile(loop=_, n=4, max_innermost_factor=64), 
_, _, _, _ = sch.split(loop=_, factors=[_, _, _, _], 
preserve_unit_iters=True, disable_predication=False), 
_, _, _, _ = sch.sample_perfect_tile(loop=_, n=4, max_innermost_factor=64), 
_, _, _, _ = sch.split(loop=_, factors=[_, _, _, _], 
preserve_unit_iters=True, disable_predication=False), 
_, _ = sch.sample_perfect_tile(loop=_, n=2, max_innermost_factor=64), 
_, _ = sch.split(loop=_, factors=[_, _], 
preserve_unit_iters=True, disable_predication=False), 
sch.reorder(_, _, _, _, _, _, _, _, _, _), 
```

The instructions in `design_spaces` don't contain concrete tile sizes and other values. They represent a template or sketch of scheduling rather than a fully instantiated schedule.

The placeholder values (represented by `_`) will be filled in during the actual schedule `sampling` process. This is intentional in TVM's meta-schedule design:

1. The design space defines the structure of possible transformations (like tiling, splitting, reordering)

2. The actual values (loop names, tile sizes, etc.) are determined during `sampling`when:
- The trace is applied to a specific module

- Random sampling fills in concrete values for the placeholders

- Postprocessors validate and refine the schedule

```cpp
std::vector<Schedule> EvolutionarySearchNode::State::SampleInitPopulation(int num) {
    //......
    std::vector<Schedule> results(num, Schedule{nullptr});
    auto f_proc_unmeasured = [this, &results, &pp](int thread_id, int trace_id) -> void {
      PerThreadData& data = this->per_thread_data_.at(thread_id);
      TRandState* rand_state = &data.rand_state;
      const IRModule& mod = data.mod;
      Schedule& result = results.at(trace_id);
      ICHECK(!result.defined());
      int design_space_index = tir::SampleInt(rand_state, 0, design_spaces.size());

      // `trace` represents an ordered sequence of schedules, which has lots
      // of placeholders of tile_size, parallel_size and so on.
      tir::Trace trace(design_spaces[design_space_index]->insts, {});
      // Apply `trace` to `mod` with the random state.
      if (Optional<Schedule> sch = pp.Apply(mod, trace, rand_state)) {
        result = sch.value();
      }
    };

/*! \note Launch `num` threads, each executing `f_proc_unmeasured` to generate one unmeasured candidate.
Total: `num` unmeasured candidates.*/
    support::parallel_for_dynamic(0, num, self->ctx_->num_threads, f_proc_unmeasured);
//......
}
```

### EvolveWithCostModel

At the stage of  `EvolveWithCostModel` , a random mutator is applied to every IRModule of population(e.g. 256 IRModule) to get a new population. After several times `evolution`(set by `genetic_num_iters`), select the top 10 performant candidate and set to the `Builder` and `Runner`.

`evolutionary_search.cc` Line 539 `EvolutionarySearchNode::State::EvolveWithCostModel`

```cpp
//......
for (int iter = 0;; ++iter) {
    // Predict normalized score with the cost model,
    std::vector<double> scores =
        PredictNormalizedScore(population, GetRef<TuneContext>(self->ctx_), this->cost_model_);

    {
      
      for (int i = 0, n = population.size(); i < n; ++i) {
        Schedule sch = population.at(i);
        IRModule mod = sch->mod();
        size_t shash = ModuleHash(mod);
        double score = scores.at(i);
        if (!exists.Has(mod, shash)) {
             // `exists` is a set of IRModule, which is used to 
            //deduplicate the IRModuels.
          exists.Add(mod, shash);
            // `heap` has the fixed size(e.g. 10), which means the worst
            // one will be removed when `heap` is full.
          heap.Push(sch, score);
        }
      }
```



```cpp
// Set the sampler of every thread, with probability from predicated normalized throughput
      for (PerThreadData& data : this->per_thread_data_) {
        data.Set(scores, self->genetic_mutate_prob, self->mutator_probs_);
      }
```



```cpp
  /*!
   * \brief Set the value for the trace and mutator samplers per thread.
   * \param scores The predicted score for the given samples.
   * \param genetic_mutate_prob The probability of mutation.
   * \param mutator_probs The probability of each mutator as a dict.
   */
  void Set(const std::vector<double>& scores, double genetic_mutate_prob,
           const Map<Mutator, FloatImm>& mutator_probs)  
   {
    /*!
     * \note Just simply think that `trace_sampler` is a function
     * returning random index of trace to be used.
     * \note `mutator_sampler` is a function returning the random-selected
     * mutator function.
   */
  trace_sampler = tir::MakeMultinomialSampler(&rand_state, scores);
    mutator_sampler = MakeMutatorSampler(genetic_mutate_prob, mutator_probs, &rand_state);
  }
```

> We can print all the builtin mutators.
> 
> ```shell
> (gdb) call tvm::Dump(self->mutator_probs_)
> {meta_schedule.MutateParallel(0x55c6b8473d48): T.float64(0.02), 
> meta_schedule.MutateUnroll(0x55c6b839a6c8): T.float64(0.029999999999999999), 
> meta_schedule.MutateComputeLocation(0x55c6b844a838): T.float64(0.050000000000000003), 
> meta_schedule.MutateTileSize(0x55c6b835fe18): T.float64(0.90000000000000002)}
> ```



Evolution from `population` to `new_population`

```cpp
auto f_find_candidate = [&cbmask, &population, &next_population, &pp, this](int thread_id,
                                                                                  int trace_id) {
        // Prepare samplers
        PerThreadData& data = this->per_thread_data_.at(thread_id);
        TRandState* rand_state = &data.rand_state;
        const IRModule& mod = data.mod;
        std::function<int()>& trace_sampler = data.trace_sampler;
        std::function<Optional<Mutator>()>& mutator_sampler = data.mutator_sampler;
        Schedule& result = next_population.at(trace_id);
        int sampled_trace_id = -1;
        // Loop until success
        for (int fail_count = 0; fail_count <= self->genetic_max_fail_count; ++fail_count) {
          /*! Get the random trace*/
            sampled_trace_id = trace_sampler();
          tir::Trace trace = population.at(sampled_trace_id)->trace().value();
          
            // Get the random mutator
            if (Optional<Mutator> opt_mutator = mutator_sampler()) {
            
            Mutator mutator = opt_mutator.value();
            
            if (Optional<tir::Trace> new_trace = mutator->Apply(trace, rand_state)) {
                  // Apply the trace and mutator to the IRModule
                if (Optional<Schedule> sch = pp.Apply(mod, new_trace.value(), rand_state)) {
                // note that sch's trace is different from new_trace
                // because it contains post-processing information
                result = sch.value();
                break;
              }
            }
          } else if (cbmask.QueryAndMark(sampled_trace_id)) {
            // Decision: do not mutate
            break;
          }
        }
        // if retry count exceeds the limit, reuse an old sample
        if (!result.defined()) {
          result = population.at(sampled_trace_id);
        }
      };
        // A multi-thread function to launch evolution
      support::parallel_for_dynamic(0, self->population_size, self->ctx_->num_threads,f_find_candidate);

      
      population.swap(next_population);
```

Above process will be launched `self->genetic_num_iters` times. `heap` will store the top 10 performant schedule.

### TroubleShooting

`target`设为`gpu` `tuning`时，需要给`tir function`加上`tir.noalias = True`的属性：

```python
@tvm.script.ir_module
    class MyTirModule:
        @T.prim_func
        def matmul(A: T.Buffer((128, 128), "float32"), B: T.Buffer((128, 128), "float32"), C: T.Buffer((128, 128), "float32")):  # type: ignore
            T.func_attr({"global_symbol": "main", "tir.noalias": True})
            for i, j, k in T.grid(128, 128, 128):
                with T.block("C"):
                    vi, vj, vk = T.axis.remap("SSR", [i, j, k])
                    with T.init():
                        C[vi, vj] = 0
                    C[vi, vj] += A[vi, vk] * B[vk, vj]
```



* bank conflict
* zero pipeline已经把所有tir binding thread了