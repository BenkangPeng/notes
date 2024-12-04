# [MLIR — Running and Testing a Lowering](https://www.jeremykun.com/2023/08/10/mlir-running-and-testing-a-lowering/)

```cpp
func.func @main(%arg0: i32) -> i32 {
  %0 = math.ctlz %arg0 : i32
  func.return %0 : i32
}
```

`math`是一个`dialect`，`ctlz`是方言`math`的`operation`

`ctlz`的语法由`dialect` 定义的`parser`所定义，`operation`的定义都有一个固定的模板。

> The rest of the syntax of the operation is determined by a parser defined by the dialect, and so many operations will have different syntaxes, though many are pulled from a fixed set of options we’ll see later in the series.

`block` ：块内的代码是顺序执行的，没有分支或跳转指令(`goto`,`break`,`continue`)会将控制流引入或带出块的中间部分。例如`if`分支的条件跳转改变了控制流，编译器可能会将这个`if`语句分成两个基本块。

`region`是一组基本块的集合，通常用于表示程序中的控制流结构，如循环或条件语句。每个区域可以包含一个或多个基本块，这些块按照特定的顺序执行。



# [MLIR — Writing Our First Pass](https://www.jeremykun.com/2023/08/10/mlir-writing-our-first-pass/)

**项目文件结构**

代码库的结构在这里是一个相当挑剔的问题。一个典型的MLIR代码库似乎将代码分成两个目录，这两个目录具有大致相等的层级结构：一个`include/`目录用于存放**头文件**和**tablegen文件**，以及一个`lib/`目录用于存放实现代码。然后，在这两个目录中，项目会有`Transform/`子目录，用于存储在某个方言内部转换代码的传递文件，`Conversion/`用于存放转换不同方言之间的传递，`Analysis/`用于存放分析传递等。这些目录中的每一个可能都有子目录，用于它们操作的具体方言。这个教程将`include/`和`lib/`合在一起没做区分：

```shell
.
├── README.md
├── WORKSPACE
├── bazel
│   └──  . . .
├── lib
│   └── Transform
│       └── Affine
│           ├── AffineFullUnroll.cpp
│           ├── AffineFullUnroll.h
│           └── BUILD
├── tests
│   └── . . .
└── tools
    ├── BUILD
    └── tutorial-opt.cpp
```



```mlir
func.func @sum_buffer(%buffer: memref<4xi32>) -> (i32) {
  %sum_0 = arith.constant 0 : i32
  %sum = affine.for %i = 0 to 4 iter_args(%sum_iter = %sum_0) -> i32 {
    %t = affine.load %buffer[%i] : memref<4xi32> 
    # 在每次迭代中，从%buffer中加载索引%i处的元素
    %sum_next = arith.addi %sum_iter, %t : i32
    affine.yield %sum_next : i32
  }
  return %sum : i32
}
```

* `memref<4xi32>` 包含4个32位整数的内存引用

* `affine.yield %sum_next : i32`表示将`%sum_next`传递给`%sum_iter`作为下一次迭代的参数



实现`pass`的方式之一是用C++定义：

主要方法是在`OperationPass`基类中定义需要的`pass`函数，定义的`pass`函数会将对应的操作(优化)锚定(`anchor`)在特定的上下文中。

```cpp
// lib/Transform/Affine/AffineFullUnroll.h
class AffineFullUnrollPass
    : public PassWrapper<AffineFullUnrollPass,
                         OperationPass<mlir::func::FuncOp>> {
private:
  void runOnOperation() override;  // implemented in AffineFullUnroll.cpp

  StringRef getArgument() const final { return "affine-full-unroll"; }

  StringRef getDescription() const final {
    return "Fully unroll all affine loops";
  }
};
```

- `runOnOperation`: 定义`pass`逻辑
- `getArgument`: the CLI argument for an `mlir-opt`-like tool.传入命令行参数
- `getDescription`: the CLI description when running `--help` on the `mlir-opt`-like tool.

```cpp
// tools/tutorial-opt.cpp
#include "lib/Transform/Affine/AffineFullUnroll.h"//自定义完全展开仿射循环
#include "mlir/include/mlir/InitAllDialects.h"//提供了注册所有 MLIR 方言（dialects）的功能
#include "mlir/include/mlir/Pass/PassManager.h"
//提供了传递管理器（pass manager）的定义。传递管理器用于管理和执行一系列的传递。
#include "mlir/include/mlir/Pass/PassRegistry.h"
//提供了传递注册表（pass registry）的定义。传递注册表用于注册和查找传递
#include "mlir/include/mlir/Tools/mlir-opt/MlirOptMain.h"
//提供了 MlirOptMain 函数的定义。MlirOptMain 是一个主函数，用于处理命令行参数并运行 MLIR 优化工具。

int main(int argc, char **argv) {
  mlir::DialectRegistry registry;
//创建一个 mlir::DialectRegistry 对象 registry。DialectRegistry 用于注册和管理 MLIR 方言
  mlir::registerAllDialects(registry);
//调用 mlir::registerAllDialects 函数，将所有已知的 MLIR 方言注册到 registry 中
  mlir::PassRegistration<mlir::tutorial::AffineFullUnrollPass>();
//注册自定义的传递 mlir::tutorial::AffineFullUnrollPass。PassRegistration 模板类用于自动注册传递，使得传递可以在传递管理器中被找到和使用

  return mlir::asMainReturnCode(
      mlir::MlirOptMain(argc, argv, "Tutorial Pass Driver", registry));
 //调用 mlir::MlirOptMain 函数，传入命令行参数、程序名称和方言注册表 registry。MlirOptMain 函数会解析命令行参数，加载 MLIR 模块，并应用指定的传递。
//mlir::asMainReturnCode 函数将 MlirOptMain 的返回值转换为适当的退出码，以便 main 函数返回给操作系统
}
```



定义`AffineFullUnrollPass`的`runOnOperation`方法，上面说了`runOnOperation`是一个`pass`的运行逻辑(logic):

```cpp
void AffineFullUnrollPass::runOnOperation() {
  getOperation().walk([&](AffineForOp op) {
    if (failed(loopUnrollFull(op))) {
      op.emitError("unrolling failed");
      signalPassFailure();
    }
  });
}
```

<del>模式重写引擎(`pattern rewrite engine`)代码看不懂先放着</del>

**Pattern Rewrite Engine**

上述方法是通过**遍历AST**对`AffineForOp`应用`loopUnrollFull()`来实现循环展开。但MLIR中还可以通过模式重写引擎进行循环展开。

> 重写模式（rewrite pattern）是OpRewritePattern的一个子类，它有一个名为matchAndRewrite的方法，该方法执行实际的转换操作。这意味着，当模式与IR中的某个结构匹配时，matchAndRewrite方法会被调用，以应用定义的转换。这种方法的好处是它可以自动处理多次应用相同转换的情况，并且开发者不需要编写大量的代码来手动遍历AST。

↑ 不懂为什么相较于遍历AST，模式重写不需要编写大量代码来手动遍历AST。

```cpp
struct AffineFullUnrollPattern ://定义模式AffineFullUnrollPattern，继承自OpRewritePattern,处理AffineForOp
  public OpRewritePattern<AffineForOp> {
  AffineFullUnrollPattern(mlir::MLIRContext *context)
      : OpRewritePattern<AffineForOp>(context, /*benefit=*/1) {}
      //benefit表示模式优先级

  LogicalResult matchAndRewrite(AffineForOp op,//匹配与重写:op与rewriter匹配时，调用loopUnrollFull展开循环
                                PatternRewriter &rewriter) const override {
    return loopUnrollFull(op);
  }
};

// A pass that invokes the pattern rewrite engine.
void AffineFullUnrollPassAsPatternRewrite::runOnOperation() {//定义runOnOperation
  mlir::RewritePatternSet patterns(&getContext());
  patterns.add<AffineFullUnrollPattern>(&getContext());//patterns添加模式AffineFullUnrollPattern
  // One could use GreedyRewriteConfig here to slightly tweak the behavior of
  // the pattern application.
  (void)applyPatternsAndFoldGreedily(getOperation(), std::move(patterns));//应用模式
}
```

**A proper greedy RewritePattern**

出于乘法比加法更慢且在硬件上更贵，于是设计一个模式重写，将乘法分解成加法，例如将`y = 9 * x`转换成`a = x+x; b = a+a; c = b+b; y = c+x`

* `PowerOfTwoExpand` : rewrites `y=C*x` as `y = C/2*x + C/2*x`
* `PeelFromMul` : 从不是2的幂的乘积中提取出一个加法，例如`y = 9*x` -> `y = 8*x + x`



`PowerOfTwoExpand` ：

```cpp
LogicalResult matchAndRewrite(
       MulIOp op, PatternRewriter &rewriter) const override {
    Value lhs = op.getOperand(0);

    // canonicalization patterns ensure the constant is on the right, if there is a constant
    // See https://mlir.llvm.org/docs/Canonicalization/#globally-applied-rules
    Value rhs = op.getOperand(1);
    auto rhsDefiningOp = rhs.getDefiningOp<arith::ConstantIntOp>();
    //获取 rhs 操作数的定义操作，并检查它是否是一个 arith::ConstantIntOp,如果不是，rhsDefiningOp = nullptr
   
    if (!rhsDefiningOp) {//rhs不是一个常数，报错
      return failure();
    }

    int64_t value = rhsDefiningOp.value();//得到rhs的值
    bool is_power_of_two = (value & (value - 1)) == 0;//判断value是否为2的幂

    if (!is_power_of_two) {
      return failure();
    }

    ConstantOp newConstant = rewriter.create<ConstantOp>(
        rhsDefiningOp.getLoc(), rewriter.getIntegerAttr(rhs.getType(), value / 2));
    //重写、新建一个常量ConstantOp ; rhsDefiningOp.getLoc()获得位置信息
    //rewriter.getIntegerAttr(rhs.getType(), value / 2)创建一个整数类型，其类型与rhs一致，值为value / 2
    
    MulIOp newMul = rewriter.create<MulIOp>(op.getLoc(), lhs, newConstant);
    AddIOp newAdd = rewriter.create<AddIOp>(op.getLoc(), newMul, newMul);
    //新建乘法、加法

    rewriter.replaceOp(op, {newAdd});//newAdd替换op
    rewriter.eraseOp(rhsDefiningOp);

    return success();
  }
```

`PeelFromMul`

```cpp
LogicalResult matchAndRewrite(MulIOp op,
                                PatternRewriter &rewriter) const override {
    Value lhs = op.getOperand(0);
    Value rhs = op.getOperand(1);
    auto rhsDefiningOp = rhs.getDefiningOp<arith::ConstantIntOp>();
    if (!rhsDefiningOp) { return failure(); }
    int64_t value = rhsDefiningOp.value();

    // We are guaranteed `value` is not a power of two, because the greedy
    // rewrite engine ensures the PowerOfTwoExpand pattern is run first, since
    // it has higher benefit.

    ConstantOp newConstant = rewriter.create<ConstantOp>(
        rhsDefiningOp.getLoc(), rewriter.getIntegerAttr(rhs.getType(), value - 1));
    MulIOp newMul = rewriter.create<MulIOp>(op.getLoc(), lhs, newConstant);
    AddIOp newAdd = rewriter.create<AddIOp>(op.getLoc(), newMul, lhs);

    rewriter.replaceOp(op, {newAdd});
    rewriter.eraseOp(rhsDefiningOp);
```



# [MLIR — Using Tablegen for Passes](https://www.jeremykun.com/2023/08/10/mlir-using-tablegen-for-passes/)

好家伙，一上来就跟我说，写`pass`一般不像上一章用C++调Api来实现，而是用`tablegen`。感情我白看了，而且模式重写引擎的代码也不是很好懂。



```cpp
def AffineFullUnroll : Pass<"affine-full-unroll"> {
  let summary = "Fully unroll all affine loops";
  let description = [{
    Fully unroll all affine loops. (could add more docs here like code examples)
  }];
  let dependentDialects = ["mlir::affine::AffineDialect"];
}
```

* `tablegen`以`def`关键字开头。需要注意的是：`def`与`class`在`tablegen`中是不同的。`def`表明接下来的代码是要被`tablegen`生成实际的代码；而`class`定义的是一个非被实例的模板，不会被tablegen生成对应代码，允许在多处实例化与重用。简而言之，def 是用于生成代码的，而 class 是用于定义模板和重用代码的。

* 使用`tablegen`生成的`Passes.h.inc`见[gist link](https://gist.github.com/j2kun/4d21d5bc85f6c9326bef81eaf6c323e7) 生成的C++代码风格都是`#ifdef……#endif`,即用`宏`为参数，`#include`为函数组织起来的(`this pattern of using #include as a function with #define as the argument`)
* 所以在`AffineFullUnroll.h`和`AffineFullUnrollPatternRewrite.h`中，通过声明宏`GEN_PASS_DECL_AFFINEFULLUNROLL` `GEN_PASS_DECL_AFFINEFULLUNROLLPATTERNREWRITE`来开启这两个`Pass`的定义(定义了这两个宏，`#include Passes.h.inc`中就可以将这两个`Pass`的定义`include`进来。见(见[gist link](https://gist.github.com/j2kun/4d21d5bc85f6c9326bef81eaf6c323e7) Line14，87)。

* 在`Passes.h`中定义`GEN_PASS_REGISTRATION`来开启所有`Pass`(包括以上两个Pass)的注册。见[gist link](https://gist.github.com/j2kun/4d21d5bc85f6c9326bef81eaf6c323e7) Line155，在`Passes.h`中定义了`GEN_PASS_REGISTRATION`,那么`#include Passes.h.inc`中就会创建`registerAffineFullUnroll`、`registerAffineFullUnrollPatternRewrite`.

* 综上，`lib/Transform/Affine`文件夹层次结构就很清楚了：`Passes.td`中定义了 `AffineFullUnroll` `AffineFullUnrollPatternRewrite`这两个`Pass` , 生成的`build/lib/Tranform/Affine/Passes.h.inc`中包含了这两个`Pass`的**定义**和**注册**。`AffineFullUnroll.h`中通过定义宏`GEN_PASS_DECL_AFFINEFULLUNROLL`,在`#include Passes.h.inc`中引入了`AffineFullUnroll`的**定义**和**注册**。`AffineFullUnrollPatternRewrite.h`同理。

  现在`AffineFullUnroll.h`和`AffineFullUnrollPatternRewrite.h`中包含了这两个Pass的定义与注册。   `Pass.h`中`#include`进了这两个文件：

  ```cpp
  #include "lib/Transform/Affine/AffineFullUnroll.h"
  #include "lib/Transform/Affine/AffineFullUnrollPatternRewrite.h"
  ```

  然后定义宏`GEN_PASS_REGISTRATION`,开启了这两个`Pass`的注册。

* 生成的`Passes.h.inc`中定义了前面提到的一个`Pass`所需要定义的`getArgument`、`getDescription()`函数，但缺少函数`RunOnOperation`的定义(即`Pass logic`的定义)
  * 好吧，上面的说法有些偏差：`Passes.h.inc`中并没有`AffineFullUnroll`的**定义**，而是定义了`AffineFullUnrollBase`。`AffineFullUnroll`的定义在`AffineFullUnroll.cpp`中通过继承`AffineFullUnrollBase`来定义结构体`AffineFullUnroll`   (`AffineFullUnrollPatternRewrite`同理)。`AffineFullUnroll`的定义和`runOnOperation`都在`AffineFullUnroll.cpp`中实现：

```cpp
struct AffineFullUnroll : impl::AffineFullUnrollBase<AffineFullUnroll> {
  using AffineFullUnrollBase::AffineFullUnrollBase;//使用AffineFullUnrollBase的构造函数

  void runOnOperation() {
    getOperation()->walk([&](AffineForOp op) {
      if (failed(loopUnrollFull(op))) {
        op.emitError("unrolling failed");
        signalPassFailure();
      }
    });
  }
};
```

# [MLIR — Defining a New Dialect](https://www.jeremykun.com/2023/08/21/mlir-defining-a-new-dialect/)

`dialect`使用`tablegen`定义：`PolyDialect.td`:

```tablegen
include "mlir/IR/DialectBase.td"

def Poly_Dialect : Dialect {
  let name = "poly";
  let summary = "A dialect for polynomial math";
  let description = [{
    The poly dialect defines types and operations for single-variable
    polynomials over integers.
  }];

let cppNamespace = "::mlir::tutorial::poly";
}
```

`dialect`定义的方式与`pass`差不多，但基于`PolyOps.td`文件生成了多个目标文件，见`cmakelist`：

```cmake
set(LLVM_TARGET_DEFINITIONS PolyOps.td)
mlir_tablegen(PolyOps.h.inc -gen-op-decls)
mlir_tablegen(PolyOps.cpp.inc -gen-op-defs)
mlir_tablegen(PolyTypes.h.inc -gen-typedef-decls -typedefs-dialect=poly)
mlir_tablegen(PolyTypes.cpp.inc -gen-typedef-defs -typedefs-dialect=poly)
mlir_tablegen(PolyDialect.h.inc -gen-dialect-decls -dialect=poly)
mlir_tablegen(PolyDialect.cpp.inc -gen-dialect-defs -dialect=poly)
```

此外，`dialect`也需要在`tutorial-opt.cpp`中额外注册：

```cpp
mlir::DialectRegistry registry;
registry.insert<mlir::tutorial::poly::PolyDialect>();
mlir::registerAllDialects(registry);
```

**一种方言的文件组织形式(以poly方言为例)**:

* `PolyTypes.h`只包含`PolyTypes.h.inc`
* 在 `PolyDialect.cpp` 中包含 `PolyTypes.cpp.inc`
* **添加 PolyTypes.cpp**：如果需要，应该添加一个 PolyTypes.cpp 文件，这个文件包含了一些额外的实现，这些实现是 tablegen（一个代码生成工具）声明的，但是不能自动生成的函数。(项目文件中并没有该文件，但在`PolyOps.cpp`中实现了一些不能自动生成的逻辑。)

* 总之，每一个`.h`文件对应一个`.h.inc`，每一个`.cpp`对应`.cpp.inc`，并且在`PolyDialect.cpp`中将它们连接起来。



以上定义的`poly`方言是空的，接下来为它加一些内容：

**添加一个poly type**

```tablegen
// PolyTypes.td
#ifndef LIB_DIALECT_POLY_POLYTYPES_TD_
#define LIB_DIALECT_POLY_POLYTYPES_TD_

include "PolyDialect.td"
include "mlir/IR/AttrTypeBase.td"

// A base class for all types in this dialect
class Poly_Type<string name, string typeMnemonic> : TypeDef<Poly_Dialect, name> {
  let mnemonic = typeMnemonic;
}

def Polynomial : Poly_Type<"Polynomial", "poly"> {
  let summary = "A polynomial with u32 coefficients";

  let description = [{
    A type for polynomials with integer coefficients in a single-variable polynomial ring.
  }];
  //单变量多项式

  let parameters = (ins "int":$degreeBound);//输入参数是$degreeBound,类型是int，表示多项式的最高次数
  let assemblyFormat = "`<` $degreeBound `>`";
}

#endif  // LIB_DIALECT_POLY_POLYTYPES_TD_
```

**添加操作(Operations)**

```tablegen
class Poly_BinOp<string mnemonic> : Op<Poly_Dialect, mnemonic, [Pure, ElementwiseMappable, SameOperandsAndResultType]> {
  //                                  ↑属于Poly_Dialect方言，助记符，pure表示操作无副作用？，
  //              ElementwiseMappable表示可以逐元素操作，SameOperandsAndResultType表示操作数和结果类型相同
  let arguments = (ins PolyOrContainer:$lhs, PolyOrContainer:$rhs);
  //        接受两个输入参数，类型为PolyOrContainer，名称为lhs和rhs
  let results = (outs PolyOrContainer:$output);
  let assemblyFormat = "$lhs `,` $rhs attr-dict `:` qualified(type($output))";
  //例如 ：%result = poly.add %lhs, %rhs, {attributeName = attributeValue} : poly-or-container
  let hasCanonicalizer = 1;//支持规范化，即支持特定规则对多项式进行简化
  let hasFolder = 1;//支持合并(折叠)优化
  let hasCanonicalizer = 1;//支持规范化，即支持特定规则对多项式进行简化
}

def Poly_AddOp : Poly_BinOp<"add"> {
  let summary = "Addition operation between polynomials.";
}

def Poly_SubOp : Poly_BinOp<"sub"> {
  let summary = "Subtraction operation between polynomials.";
}

def Poly_MulOp : Poly_BinOp<"mul"> {
  let summary = "Multiplication operation between polynomials.";
}
```



# [MLIR — Using Traits](https://www.jeremykun.com/2023/09/07/mlir-using-traits/)

* **traits and interfaces(特性与接口)**

`interfaces`好理解，就是将一系列方法、属性封装成类，基于派生、继承、多态等手段，调用抽象的接口。

<del>`traits`是将**类型**关联起来，从而实现类型特征的抽象。例如`C++`中的`template`</del>

好吧，`mlir`语境下，`traits`是指用来描述和约束操作（操作是 MLIR 的基本构建块之一）的特性和行为的特性或属性集。相比于传统的 C++ 或其他编程语言中，MLIR 的 traits 主要用于在编译器级别进行优化和生成代码时提供特定的语义信息。例如`pure`表示操作时无副作用的，`ConstantLike`表示该操作的输出可视为常量。

**Traits and Loop Invariant Code Motion(循环不变量外提)**

循环不变量外提是mlir提供的一个pass，用于将循环中的不变量指令移动到循环外计算，从而减少无效指令的运行。但并不是所有含不变量的指令都能够外提，**外提**需要满足两个条件：①指令是`NoMemoryEffect`，也就是说，指令不对写内存产生影响。(`the operation does not have any side effects related to writing to memory`) ② `AlwaysSpeculatable` 指令可以被编译器推断执行。

`MLIR`提供了一个`Trait`叫`Pure`,它包含了`NoMemoryEffect`和`AlwaysSpeculatable`,我们可以直接给类型的模板(class)添加这个trait：

```tablegen
class Poly_BinOp<string mnemonic> : Op<Poly_Dialect, mnemonic, [Pure]> {
```

添加完这些Trait后，tablegen生成的文件中也增加了以下内容

```cpp
// PolyOps.h.inc
class SubOp : public ::mlir::Op<SubOp,
::mlir::OpTrait::ZeroRegions,
::mlir::OpTrait::OneResult,
::mlir::OpTrait::OneTypedResult<::mlir::tutorial::poly::PolynomialType>::Impl,
::mlir::OpTrait::ZeroSuccessors,
::mlir::OpTrait::NOperands<2>::Impl,
::mlir::OpTrait::OpInvariants,
::mlir::ConditionallySpeculatable::Trait,            // <-- new
::mlir::OpTrait::AlwaysSpeculatableImplTrait,   // <-- new
::mlir::MemoryEffectOpInterface::Trait>          // <--- new
{ ... }
```

```cpp
// PolyOps.h.inc
void SubOp::getEffects(
  ::llvm::SmallVectorImpl<::mlir::SideEffects::EffectInstance<::mlir::MemoryEffects::Effect>> &effects) {
}
```

接下来就是讲了一些`Traits` , 例如：`ElementwiseMappable`、`SameOperandsAndResultElementType`、`Involution`、`Idempotent`等，这里没抄了。

# MLIR — Folders and Constant Propagation

**Constant Propagation vs Canonicalization**

常量传播(`Constant Propagation`)：当程序中，一个变量(或一个`operation`)的返回/输出总是一个常量时，可以在之后的程序中用常量替代变量，从而减少编译计算。

规范化(`canonicalize`)也能进行常量折叠，甚至删除死代码(`Constant Propagation`不会删除)。

以下是一个例子：

```mlir
//  tests/scc[p.mlir
func.func @test_arith_sccp() -> i32 {
  %0 = arith.constant 7 : i32
  %1 = arith.constant 8 : i32
  %2 = arith.addi %0, %0 : i32
  %3 = arith.muli %0, %0 : i32
  %4 = arith.addi %2, %3 : i32
  return %2 : i32
}
```

**tutorial-opt --sccp sccp.mlir**

```mlir
func.func @test_arith_sccp() -> i32 {
  %c63_i32 = arith.constant 63 : i32
  %c49_i32 = arith.constant 49 : i32
  %c14_i32 = arith.constant 14 : i32
  %c8_i32 = arith.constant 8 : i32
  %c7_i32 = arith.constant 7 : i32
  return %c14_i32 : i32
}
```

**tutorial-opt --canonicalize sccp.mlir**

```mlir
func.func @test_arith_sccp() -> i32 {
  %c14_i32 = arith.constant 14 : i32
  return %c14_i32 : i32
}
```

`--sccp`和`--canonicalize`都是通过`folding`实现的。

为了支持`Folding`操作，我们需要完善：

* 完善`constant operation`
* 添加`materialization hook`
* 为每个`operation`添加`folder`

**constant operation**

之前我们声明一个`poly.poly`类型的量，需要这样两步`SSA`声明：

```mlir
%0 = arith.constant dense<[1, 2, 3]> : tensor<3xi32>
%p0 = poly.from_tensor %0 : tensor<3xi32> -> !poly.poly<10>
```

现在我们想要简化成：

```mlir
%0 = poly.constant dense<[2, 8, 20, 24, 18]> : !poly.poly<10>
```

给`constant operation`添加`ConstantLike trait` , 修改`arguments`和`assemblyFormat` :

```mlir
def Poly_ConstantOp : Op<Poly_Dialect, "constant", [Pure, ConstantLike]> {   // new
  let summary = "Define a constant polynomial via an attribute.";
  let arguments = (ins AnyIntElementsAttr:$coefficients);    // new
  let results = (outs Polynomial:$output);
  let assemblyFormat = "$coefficients attr-dict `:` type($output)";//new
}
```

**Adding folder**

```mlir
let hasFolder = 1;
```

并在`PolyOps.cpp`中定义各`Operaion`的`::fold`函数

# [MLIR — Verifiers](https://www.jeremykun.com/2023/09/13/mlir-verifiers/)

**Trait-based Verifiers**

这类`verifies`有着`traits`一样的格式，都是定义在`operation`的定义参数内。

例如`Poly_BinOp`的`SameOperandsAndResultType`,它会去验证(推断)输入输出类型是否相同。以下是Poly_BinOp的前后变化

```git
- class Poly_BinOp<string mnemonic> : Op<Poly_Dialect, mnemonic, [Pure, ElementwiseMappable]> {
+ class Poly_BinOp<string mnemonic> : Op<Poly_Dialect, mnemonic, [Pure, ElementwiseMappable, SameOperandsAndResultType]> {
  let arguments = (ins PolyOrContainer:$lhs, PolyOrContainer:$rhs);
  let results = (outs PolyOrContainer:$output);
- let assemblyFormat = "$lhs `,` $rhs attr-dict `:` `(` qualified(type($lhs)) `,` qualified(type($rhs)) `)` `->` qualified(type($output))";
+ let assemblyFormat = "$lhs `,` $rhs attr-dict `:` qualified(type($output))";#无需指明输入的类型
  let hasFolder = 1;
}
```

对于操作`Poly_EvalOp`，输入输出一定是不同的，可以使用`Verifies`  `AllTypesMatch`:

```mlir
def Poly_EvalOp : Op<Poly_Dialect, "eval", [AllTypesMatch<["point", "output"]>]> {
  let summary = "Evaluates a Polynomial at a given input value.";
  let arguments = (ins Polynomial:$input, AnyInteger:$point);
  let results = (outs AnyInteger:$output);
  let assemblyFormat = "$input `,` $point attr-dict `:` `(` qualified(type($input)) `,` type($point) `)` `->` type($output)";
}
```





**A custom verifier**

我们可以自己定义`verifier`

```tablegen
// PolyOps.td
// Inject verification that all integer-like arguments are 32-bits
def Has32BitArguments : NativeOpTrait<"Has32BitArguments"> {
  let cppNamespace = "::mlir::tutorial::poly";
}
```

然后在`PolyTraits.h`中实现`Has32BitArguments`

```cpp
template <typename ConcreteType>
class Has32BitArguments : public OpTrait::TraitBase<ConcreteType, Has32BitArguments> {
 public:
  static LogicalResult verifyTrait(Operation *op) {
    for (auto type : op->getOperandTypes()) {
      // OK to skip non-integer operand types
      if (!type.isIntOrIndex()) continue;

      if (!type.isInteger(32)) {
        return op->emitOpError()
               << "requires each numeric operand to be a 32-bit integer";
      }
    }

    return success();
  }
};
```

# [MLIR — Canonicalizers and Declarative Rewrite Patterns](https://www.jeremykun.com/2023/09/20/mlir-canonicalizers-and-declarative-rewrite-patterns/)

待看，先去看mlir toy tutorial了





# [Chapter 1: Toy Language and AST](https://mlir.llvm.org/docs/Tutorials/Toy/Ch-1/)

第一章做了Toy语言的词法分析、语法分析并生成了AST。

`Lexer.h`文件定义了`Lexer`类，用于遍历代码字符串，返回各个词的词法。

`AST.h`定义各个语法树结点，例如`NumberExprAST`、`LiteralExprAST`

`Parser.h`解析代码字符串，根据词法，实例化语法树结点。

# Chapter 2: Esmitting Basic MLIR

第二章讲的主要是`Dialect`的定义，这些都在[mlir-tutorial](https://github.com/j2kun/mlir-tutorial)详尽地描述过，此处不赘述。

我更关心`mlir-generator`，它在`Dialect.cpp`和`MLIRGen.cpp`中实现，但第二章没做任何讲述。

值得注意的是，第二章的`Dialect.cpp`中对操作定义了多个函数，例如`AddOp::build`、`AddOp::parse`、`AddOp::print`，这些在`mlir-tutorial-j2kun`中没有提到，重点来看这些内容。

**Operation::build**

Operation的build方法可以在`tablegen`中定义并自动生成：



MLIRGEN.cpp中的builder.create()隐式调用了operation::build()方法

dump()方法隐式调用了Operation::print() ， Operation::parse()用于解析Mlir输入



如果在`Ops.td`中定义了`Operations`的`assemblyFormat`，那么`Ops.cpp.inc`会自动生成该操作的`print`和`parse`方法，而无需在`Dialect.cpp`中定义。

mlir好多源代码都是用tablegen生成的(.inc文件)

# Chapter 4 Enabling Generic Transformation with Interfaces

**==疑问==**：没有找到`CastOp`匹配`generic_call`与`callee`的类型的逻辑代码。

第三章介绍了模式匹配与重写，它需要给每一个操作定义模式重写的逻辑。当需要模式重写的操作有许多时，代码就会变得冗余。MLIR引入了`Interface`来解决这一问题，简单而言，如果多个操作具有相似的模式重写逻辑，那么就可以定义通用的`Interface`实现，再将这一个`Interface`应用于多个Operations即可。

**Inlining**

对于大多数函数而言，内联这一模式重写的逻辑是相似或一致的



# 相关资料

[HCL-Dialect作者写的介绍，放在cs6120的官网上](https://www.cs.cornell.edu/courses/cs6120/2023fa/blog/hcl-amc/)

[知乎文章解析Mlir toy tutorial](https://zhuanlan.zhihu.com/p/642136872)

[这个人的知乎有mlir教程的翻译](https://www.zhihu.com/column/c_1416415517393981440)

[nelli: A lightweight frontend for MLIR (arxiv.org)](https://arxiv.org/pdf/2307.16080)

[知乎一个学习路线](https://www.zhihu.com/question/435109274/answer/3585914452?utm_psn=1836912627655262208)

[在mlir toy tutorial的基础上添加MatMul Operation教程](https://zhuanlan.zhihu.com/p/647544289)

[一个人的llvm学习笔记](https://llvm-study-notes.readthedocs.io/en/latest/)

