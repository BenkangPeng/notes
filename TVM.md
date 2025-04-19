## LLVM&TVM构建

### LLVM CMake配置

```shell
cmake -S ../llvm -G Ninja -DLLVM_ENABLE_PROJECTS="clang;lld;mlir;clang-tools-extra;lldb;polly;cross-project-tests" -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_RTTI=ON  -DLLVM_ENABLE_ASSERTIONS=ON -DLLVM_BUILD_EXAMPLES=ON -DLLVM_TARGETS_TO_BUILD="X86;ARM;AArch64;NVPTX;AMDGPU" -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DLLVM_TEMPORARILY_ALLOW_OLD_TOOLCHAIN=ON -DLLVM_BUILD_LLVM_DYLIB=ON -DLLVM_LINK_LLVM_DYLIB=ON -DLLVM_INSTALL_UTILS=ON
```

* 注意打开`_DLLVM_ENABLE_RTTI`, `TVM`需要`RTTI`    ==<-- 哪来的消息，不确定==

`LLVM`编译完成后，检查`llvm-config`版本与位置

```shell
$ llvm-config --version
21.0.0git
$ which llvm-config

```

### TVM构建

按照`TVM`官方教程 [Install from Source — tvm 0.20.dev0 documentation](https://tvm.apache.org/docs/install/from_source.html)

编译的最后还会提醒要安装`Cython`

* `Cutlass`报错

在`build/config.cmake`加入

```
set(CMAKE_CUDA_ARCHITECTURES "native")
```

### Jupyter使用

使用`jupyter`还需要在当前`python环境`安装`ipykernel`和`jupyterlab`

`vscode python静态分析配置`：

因为`tvm`源文件不在`python`的`site-packages`中，需要将`TVM`的`python`路径加入`$PYTHONPATH`变量，可选择在`.bashrc`中全局修改，也可以在当前工程目录新建`.env`文件

**设置.env**

`.env`文件中设置`PYTHONPATH`(或者可以加到`~/.bashrc`)

```.env
PYTHONPATH=/home/<yourname>/github/tvm/python:$PYTHONPATH
```



**在jupyter中使用tvm.build将模型部署到GPU爆nvcc与gcc版本错误**

在`jupyter`中运行`!gcc --version`显示的还是`7.3`版本。将`gcc 9`加入`.env`中：

```.env
CC=/opt/rh/devtoolset-9/root/usr/bin/gcc
CXX=/opt/rh/devtoolset-9/root/usr/bin/g++
PATH=/opt/rh/devtoolset-9/root/usr/bin:${PATH}
```



`miniconda`安装不需要手动把`miniconda`加到`PATH`，安装后会自动在`.bashrc`中加：

```shell
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/<yourname>/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/<yourname>/miniconda3/etc/profile.d/conda.sh" ]; then
        . "/home/<yourname>/miniconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/<yourname>/miniconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

==.condarc换源==

```shell
$ cat ~/.condarc                                                                                                              
channels:                                                                                                                                                    
  - defaults                                                                                                                                                 
show_channel_urls: true                                                                                                                                      
default_channels:                                                                                                                                            
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main                                                                                                  
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r                                                                                                     
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2                                                                                                 
custom_channels:                                                                                                                                             
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud                                                                                           
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud  
```



## Machine Learning Compilation课程学习

[MLC | Schedule](https://mlc.ai/summer22/schedule)



### 2. Tensor Program Abstraction

一个`IRModule`示例：

```python
@tvm.script.ir_module#将一个类声明为TVM的IRModule
class MyModule:
    @T.prim_func #定义一个primitive func原语函数
    def mm_relu(A: T.Buffer((128, 128), "float32"),
                B: T.Buffer((128, 128), "float32"),
                C: T.Buffer((128, 128), "float32")):
        T.func_attr({"global_symbol": "mm_relu", "tir.noalias": True})
        Y = T.alloc_buffer((128, 128), dtype="float32")
        for i, j, k in T.grid(128, 128, 128):
            with T.block("Y"):#定义一个计算块及其名称
                vi = T.axis.spatial(128, i)#空间轴
                vj = T.axis.spatial(128, j)
                vk = T.axis.reduce(128, k) #归约轴
                with T.init():
                    Y[vi, vj] = T.float32(0)
                Y[vi, vj] = Y[vi, vj] + A[vi, vk] * B[vk, vj]
        for i, j in T.grid(128, 128):
            with T.block("C"):
                vi = T.axis.spatial(128, i)
                vj = T.axis.spatial(128, j)
                C[vi, vj] = T.max(Y[vi, vj], T.float32(0))
```

* `T.init()`的理解

`T.init()`并非在每一次`k`循环迭代时就运行一次。`TVM`会将`with T.init()`中的内容提到**归约轴**循环外层处，在每个循环迭代变量`(i,j)`处执行一次。

* 更简便的语法糖

```python
vi, vj, vk = T.axis.remap("SSR", [i, j, k])
```

* 循环变换

```python
block_Y = sch.get_block("Y", func_name="mm_relu")
i, j, k = sch.get_loops(block_Y)

j0, j1 = sch.split(j, factors=[None, 4])#将j循环分割为j0,j1;其中j0=32, j1=4
sch.reorder(j0, k, j1)#重新排序j0,k,j1
```

* 移动`block`位置

```python
block_C = sch.get_block("C", func_name="mm_relu")
sch.reverse_compute_at(block_C, j0)#将block_C放到j0循环下
sch.mod.show()

sch.decompose_reduction()
```



### 3. 

陈天奇老师的B站课程看完了前7课，先放一下，看点源码











