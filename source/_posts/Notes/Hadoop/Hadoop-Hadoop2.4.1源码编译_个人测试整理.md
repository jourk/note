---
title: Hadoop-Hadoop2.4.1源码编译_个人测试整理
tags:
  - hadoop
categories:
  - Notes
date: 2014-08-06 11:24:06
---
> 注明：本教程所有工具软件均安装在/usr/local/目录下,所以要把已下载的工具包Ftp传到生产环境的/usr/local/目录下,然后进入此目录操作。

``` bash
$ cd /usr/local
```

所用到的软件：

- JDK1.7
- Maven
- Findbugs
- Protobuf
- hadoop2.4.1-src

## 安装JDK

hadoop是java写的，编译hadoop必须安装`jdk`。

从oracle官网下载jdk-7u45-linux-x64.tar.gz。

执行以下命令解压缩jdk

``` bash
$ tar -zxvf jdk-7u45-linux-x64.tar.gz
```

会生成一个文件夹`jdk1.7.0_45`，然后设置环境变量中。

执行命令`vi/etc/profile`，增加以下内容到配置文件中。

```
export JAVA_HOME=/usr/local/jdk
export JAVA_OPTS="-Xms1024m-Xmx1024m"
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$CLASSPATH
export PATH=$JAVA_HOME/bin:$PATH
```

保存退出文件后，执行以下命令

``` bash
$ source /etc/profile
```

`java -version`看到显示的版本信息即正确。

## 安装maven

hadoop源码是使用`maven`组织管理的，必须下载maven。从maven官网下载apache-maven-3.2.2-bin.tar.gz下载，不要选择3.1。

执行以下命令解压缩

``` bash
$ tar -zxvf  apache-maven-3.2.2-bin.tar.gz
```

会生成一个文件夹`apache-maven-3.2.2`，重命名为`maven`，然后设置环境变量中。

``` bash
$ mv apache-maven-3.2.2 maven
```

执行命令`vi /etc/profile`，编辑结果如下所示

```
export MAVEN_HOME=/usr/local/maven
export PATH=${PATH}:${MAVEN_HOME}/bin
```

保存退出文件后，执行以下命令

``` bash
$ source /etc/profile
```

``` bash
$ mvn -version
```

如果看到下面的显示信息，证明配置正确了。

## 安装findbugs（可选步骤）
`findbugs`是用于`生成文档`的。如果不需要编译生成文档，可以不执行该步骤。从findbugs官网下载findbugs-3.0.0-dev-20131204-e3cbbd5.tar.gz。

执行以下命令解压缩

``` bash
$ tar -zxvf findbugs-3.0.0-dev-20131204-e3cbbd5.tar.gz
```

会生成一个文件夹`findbugs-3.0.0-dev-20131204-e3cbbd5`，重命名为`findbugs`，然后设置环境变量中。

``` bash
$ mv findbugs-3.0.0-dev-20131204-e3cbbd5 findbugs
```

修改配置文件改变镜像

``` bash
$ vi /usr/local/maven/conf/settings.xml
```

``` xml
<mirror>
  <id>nexus-osc</id>
  <mirrorOf>*</mirrorOf>
  <name>Nexusosc</name>
  <url>http://maven.oschina.net/content/groups/public/</url>
</mirror>
```

执行命令`vi /etc/profile`，编辑结果如下所示

```
export FINDBUGS_HOME=/usr/local/findbugs
export PATH=${PATH}:${FINDBUGS_HOME}/bin
```

保存退出文件后，执行以下命令

``` bash
$ source /etc/profile
```

``` bash
$ findbugs -version
```

如果看到下面的显示信息，证明配置正确了。

## 安装protoc

hadoop使用`protocol buffer`通信，从protoc官网下载protoc，选择protobuf-2.5.0.tar.gz 下载。

为了编译安装protoc，需要下载几个工具，顺序执行以下命令

``` bash
$ yum install gcc 
$ yum intall gcc-c++ 
$ yum install make
```

如果操作系统是redhat6那么gcc和make已经安装了。在命令运行时，需要用户经常输入`y`。

然后执行以下命令解压缩protobuf

``` bash
$ tar -zxvf protobuf-2.5.0.tar.gz
```

会生成一个文件夹`protobuf-2.5.0`，执行以下命令`编译`protobuf。

``` bash
$ cd protobuf-2.5.0
$ ./configure --prefix=/usr/local/protoc/ 
$ make && make install
```

只要不出错就可以了。

执行完毕后，编译后的文件位于`/usr/local/protoc/`目录下，我们设置一下环境变量

执行命令`vi /etc/profile`，编辑结果如下所示

```
export PATH=${PATH}:/usr/local/protoc/bin
```

保存退出文件后，执行以下命令

``` bash
$ source /etc/profile
```

``` bash
$ protoc --version
```

如果看到下面的显示信息，证明配置正确了。

## 安装编译依赖

顺序执行以下命令

``` bash
$ yum install cmake 
$ yum install openssl-devel 
$ yum install ncurses-devel
```

安装完毕即可。

## 编译hadoop2.4.1源码

从hadoop官网下载2.4.1稳定版，下载hadoop-2.4.1-src.tar.gz。

执行以下命令解压缩jdk

``` bash
$ tar -zxvf hadoop-2.4.1-src.tar.gz 
```

进入到目录`/usr/local/hadoop-2.4.1-src`中，执行命令

``` bash
$ mvn package -Pdist,native -DskipTests -Dtar
```

如果`没有安装findbugs`，把上面命令中的`docs去掉`即可，就不必生成文档了。

该命令会从外网下载依赖的jar，编译hadoop源码，需要花费很长时间。

显示sucess结果，就是编译成功了。恭喜！

好了，编译完成了。

编译后的代码在`hadoop-2.4.1-src/hadoop-dist/target`下面。

