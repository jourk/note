---
title: Java-Https-证书导入
tags:
  - java
categories:
  - Notes
date: 2014-06-27 06:27:06
---
>在项目开发中,有时会遇到与SSL安全证书导入打交道的，如何把证书导入java中的cacerts证书库呢？

其实很简单，方法如下：

## 第一步：下载证书  ##
* 进入https://www.xxx.com 网站
* 在该网页上右键 >> 属性 >> 点击"证书"
* 再点击上面的"详细信息"切换栏
* 再点击右下角那个"复制到文件"的按钮
* 就会弹出一个证书导出的向导对话框，按提示一步一步完成就行了。
* 例如：保存为abc.cer,放在C盘下

## 第二步：导入证书
如何把上面那步的(abc.cer)这个证书导入java中的cacerts证书库里?             
假设你的jdk安装在C:jdk1.5这个目录

1、开始 >> 运行 >> 输入cmd 进入dos命令行

2、再用cd进入到`C:\jdk1.5\jre\lib\security`这个目录下

3、敲入如下命令回车执行
```
~keytool -import -alias cacerts -keystore cacerts -file d:softwareAKAZAM-Mail.cer
```
4、此时命令行会提示你输入cacerts证书库的密码

你敲入`changeit`就行了，这是java中cacerts证书库的默认密码

你自已也可以修改的。

5、导入后用`-list`查看(没有使用-alias指定别名，所以是mykey)，其中md5会和证书的md5对应上。

mykey, 2014-06-26, trustedCertEntry,

认证指纹 (MD5)：
```
8D:A2:89:9A:E4:17:07:0B:BD:B0:0C:36:11:39:D0:3D
```

ok,大功告成！

注：以后更新时，先删除原来的证书，然后导入新的证书
```
keytool -list -keystore cacerts
keytool -delete -alias akazam_email -keystore cacerts
keytool -import -alias akazam_email -file akazam_email.cer -keystore cacerts
```