refer to [🔗installation](https://tilelang.com/get_started/Installation.html)

我使用`Method 2`使用3rdparty中的tvm来构建的tilelang.

> method 1中如果要使用自己的tvm来build tilelang，大概率不成功，因为tilelang与upstream tvm不兼容，tilelang team对tvm做了定制修改[🔗tile-ai/tvm](https://github.com/tile-ai/tvm)

我使用的构建命令如下：

```shell
git clone --recursive https://github.com/tile-ai/tilelang
cd tilelang
mkdir build
cp 3rdparty/tvm/cmake/config.cmake build
cd build
echo "set(USE_CUDA ON)" >> config.cmake 
cmake .. -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-12.4 \
    -DCUDA_NVRTC_LIBRARY=/usr/local/cuda-12.4/lib64/libnvrtc.so
make -j
```

> * 没有打开`LLVM`，因为tilelang/3rdparty/tvm使用的是很老版本的llvm，而我的环境是llvm 21.0。llvm没有backward compatibility，编译会报错。
> * 指定了CUDA位置

设置环境变量这一步，官方文档没有更新。

不仅要将`/path/to/tilelang`加入`$PYTHONPATH` ，还要设置`$TVM_LIBARARY_PATH`

我在使用miniconda3隔离了不同conda环境的`$PYTHONPATH`与`$TVM_LIBARARY_PATH`(因为我还有独立构建的tvm 0.22dev)

tilelang文档中说`The build outputs (e.g., `libtilelang.so`, `libtvm.so`, `libtvm_runtime.so`) will be generated in the `build` directory.` , 其实`libtvm.so`与`libtvm_runtime.so`放在了`build/tvm`下。

设置`$PYTHONPATH `$TVM_LIBARARY_PATH`

```shell
(tilelang) [user@localhost build]$ conda env config vars set PYTHONPATH=/path/to/tilelang/
To make your changes take effect please reactivate your environment
(tilelang) [user@localhost build]$ conda env config vars set TVM_LIBRARY_PATH=/path/to/tilelang/build/tvm/
To make your changes take effect please reactivate your environment
(tilelang) [user@localhost build]$ conda deactivate
[user@localhost build]$ conda activate tilelang
WARNING: overwriting environment variables set in the machine
overwriting variable ['TVM_LIBRARY_PATH', 'PYTHONPATH']
```



