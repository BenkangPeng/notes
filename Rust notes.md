### Rust安装

* 因为windows上配的C++环境是`Mingw64` 而不是`Visual Stdio`，所以不采用rust默认安装方式(下载一个5G+的Visual stdio).

* `windows`上`Mingw64`的配置方法：`mingw`是一个轻量级移植项目，将linux的gnu工具链一直到了windows上。可以直接下载[x86_64-13.2.0-release-posix-seh-msvcrt-rt_v11-rev1.7z](https://github.com/niXman/mingw-builds-binaries/releases),  解压后放在特定的位置，例如`D:\software\mingw64`. 将`D:\software\mingw64\bin`添加进用户变量、系统变量`PATH`中。执行`gcc --version`判断是否配置成功。

* 在`rust`官网下载`rustup-init.exe`后执行，依次选择：`2) Manually install the prerequisites` 、 `y` 、`2) Customize installation` 、输入`x86_64-pc-windows-gnu`、之后选择默认即可(一直回车)。注意选择`host triple`时不选择默认的下`x86_64-pc-windows-msvc` , 而是选择`x86_64-pc-windows-gnu` 。 使用`rust --version`    `cargo --version`验证是否安装成功。
































