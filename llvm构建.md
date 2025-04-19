```shell
$ mkdir b
$ cmake -S ../llvm -G Ninja -DLLVM_ENABLE_PROJECTS="clang;lld;mlir;clang-tools-extra;lldb;polly;cross-project-tests" -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON -DLLVM_BUILD_EXAMPLES=ON -DLLVM_TARGETS_TO_BUILD="Native;ARM;AArch64" -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DLLVM_TEMPORARILY_ALLOW_OLD_TOOLCHAIN=ON
$ cmake --build . -j #或者使用ninja
```

* `-DCMAKE_BUILD_TYPE`可以换成`Debug`，但这样编译更慢、生成的文件更大
* `-DLLVM_TARGETS_TO_BUILD`指定要构建的目标架构代码生成器。
* `-DLLDB_TEST_COMPILER=/opt/rh/devtoolset-9/root/usr/bin/g++`   `-DCMAKE_INSTALL_PREFIX=`增加这两个选项可以将`clang;lldb`等工具安装到指定目录。安装的命令：`ninja` , `ninja install`

将`LLVM bin`目录添加到`PATH`

在`bashrc`中添加：

```
export LLVM_BUILD_DIR=/home/pengbenkang/github/LLVM/build
export PATH=$PATH:$LLVM_BUILD_DIR/bin
```

这样才能找到`llvm`的`package`,例如以下两条`cmake`命令:

```cmake
find_package(MLIR REQUIRED CONFIG)
find_package(LLVM REQUIRED CONFIG)
```





