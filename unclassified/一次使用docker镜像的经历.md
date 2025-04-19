* 由于项目的需要，我要用到一个编译器，于是在网上`docker pull`该编译器的镜像下来(见[使用阿里云拉取海外源镜像](使用阿里云拉取海外源镜像.md))。
* 运行该`docker`镜像

```shell
$sudo docker images
[sudo] ICer 的密码：
REPOSITORY                                                  TAG       IMAGE ID       CREATED        SIZE
registry.cn-hangzhou.aliyuncs.com/benkangpeng/hcl-dialect   v1.0.0    2e37a38dedb3   6 months ago   16GB
ubuntu                                                      latest    ba6acccedd29   2 years ago    72.8MB

$sudo docker run --rm -it registry.cn-hangzhou.aliyuncs.com/benkangpeng/hcl-dialect:1.0.0
```

由于我用阿里云构建的海外源镜像，镜像的名字也已经发生改变，注意此时不要套用原来的名字，tag也要写正确。`--rm`表示你当前在`docker`中的所有修改都将在退出镜像后被删除，`-it`表示在当前终端提供一个`docker`交互环境。

* 将主机文件链接到docker镜像中

我需要在docker镜像中使用镜像提供的编译器去编译`allo`中的代码，因此我要将`allo`导入docker镜像中。我一开始是想要在docker镜像`git clone`这个项目，但遇到了github ssh等clone限制(该docker镜像没有ssh配置)。后面找到一种类似于将主机文件挂载到docker中的方法

* 将主机文件挂载到docker镜像

```shell
$sudo docker run --rm -it -v /home/ICer/Desktop/OnWorking/allo:/root/allo  registry.cn-hangzhou.aliyuncs.com/benka
ngpeng/hcl-dialect:v1.0.0
(hcl-dev) root@1cc83d4bdd37:~# ls
allo          cmake-3.27.9.tar.gz  llvm-project  miniconda.sh
cmake-3.27.9  hcl-dialect          miniconda
```

`-v /home/ICer/Desktop/OnWorking/allo:/root/allo`表示将`/home/ICer/Desktop/OnWorking/allo`挂载到`docker`镜像中的`/root/allo`,且在`docker`镜像中对`allo/`进行的修改都将与主机同步。

* `pip`更换阿里源

```shell
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple
pip config set install.trusted-host mirrors.aliyun.com
```

