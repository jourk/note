---
title: Scala-CentOS下安装Sbt
tags:
  - scala
categories:
  - Notes
date: 2015-06-09 11:24:06
---

>参见官网配置说明http://www.scala-sbt.org/release/tutorial/Manual-Installation.html

## 下载sbt通用平台压缩包

sbt-0.13.8.tgz
http://www.scala-sbt.org/download.html

## 建立目录，解压文件到所建立目录

``` bash
sudo mkdir /usr/local/sbt
sudo tar zxvf sbt-0.13.8.tgz -C /usr/local/
```

## 建立启动sbt的脚本文件

选定一个位置，建立启动sbt的脚本文本文件，如`/usr/local/sbt/ `目录下面新建文件名为sbt的文本文件

``` bash
cd /usr/local/sbt/
```

``` bash
vim sbt
```
在sbt文本文件中添加

```
BT_OPTS=”-Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256M”
java $SBT_OPTS -jar bin/sbt-launch.jar “$@”
```

然后按`esc`键 输入 `:wq` 保存退出，注意红色字体中的路径可以是绝对路径也可以是相对路径，只要能够正确的定位到解压的sbt文件包中的`sbt-launch.jar`文件即可

## 修改sbt文件权限

``` bash
chmod u+x sbt 
```

## 配置PATH环境变量

保证在控制台中可以使用sbt命令

``` bash
vim /ect/profile
```

在文件尾部添加如下代码后，保存退出

```
export PATH=/usr/local/sbt/:$PATH
```

使配置文件立刻生效

``` bash
source /etc/profile
```

## 测试sbt是否安装成功

第一次执行时，会下载一些文件包，然后才能正常使用，要确保联网了，安装成功后显示如下

``` bash
sbt sbt-version
```
```
[info] Set current project to sbt (in build file:/usr/local/sbt/)
[info] 0.13.8
```