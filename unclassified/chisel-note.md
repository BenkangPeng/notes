### chisel安装

* 安装`java`环境，将`java`加入系统变量`PATH`后，还需要新建一个`JAVA_HOME`的系统变量指向`java`的`HOME`目录，例如`JAVA_HOME : D:\software\Java\jdk-21`，这样运行`scala-cli`时才会找到对应的`	java`环境，否则会去自动下载`java17`，下又下载不下来，报错。**(这一条chisel手册上没提)。**
* 根据手册安装`scala-cli`  

> 有人说不要用scala 3，用scala2.x的版本。在chisel官方提供的template中是：ThisBuild / scalaVersion   := "2.13.12"。那就安装旧版本吧。

* 安装`sbt`: `sbt`是`scala`的一个构建工具，用于`scala`项目的构建、管理。

> 安装完sbt后会自动新建一个变量SBT_HOME存放sbt的家目录

* 安装`vscode`插件：`Scala Syntax(official)`  `Scala(Metals)`

* 可以将`sbt`换个源，下载得更快

[华为开源镜像站_软件开发服务_华为云 (huaweicloud.com)](https://mirrors.huaweicloud.com/mirrorDetail/5ebf85de07b41baf6d0882ac?mirrorName=sbt&catalog=language)