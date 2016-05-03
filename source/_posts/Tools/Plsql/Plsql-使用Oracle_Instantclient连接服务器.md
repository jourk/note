---
title: Plsql-使用Oracle_Instantclient连接服务器
tags:
  - plsql
categories:
  - Tools
date: 2014-07-07 11:27:06
---

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通常情况下，用PL/SQL Developer连接Oracle是需要安装Oracle客户端软件的，这也就意味着你的硬盘将被占用大约1G-2G的空间，对于Windows操作系统来说，你还会多出一些开机自启动的服务。当然对于大部分人来说，并不会在自己的机器上应用所创建的数据库，而只是希望通过他的一些配置来连接访问服务器上的数据库。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其实Oracle为我们提供了轻便的工具Oracle Instantclient package，也有人称他为“Oracle即时客户端”。使用此工具，我们就可以在不安装Oracle客户端软件的情况下访问存在于其他计算机上的数据库了。

## 第一步：下载安装包
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Oralce官方网站上下载[Oracle Instantclient Basic package](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html)。在这个页面的中部找到Instant Client，在Instant Client Downloads中选择合适的版本下载。

## 第二步：解压这个安装包

1. 下载完成后，解压压缩文件至本地某路径下，例如`c:\instantclient` 。
2. 在此路径下建立文件夹`NETWORK\ADMIN`，在ADMIN文件夹下建立`tnsnames.ora`文件，文件内容即为希望连接的数据库的TNS信息。例如:

```
 WORCL =
         (DESCRIPTION =
           (ADDRESS_LIST =
             (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.1.4)(PORT = 1521))
           )
           (CONNECT_DATA =
             (SERVICE_NAME = orcl)
           )
         )

```

## 第三步：配置pl/sql developer

启动PL/SQL Developer，在登录窗口界面,点击取消按钮就可以进行主界面,点击`Tools->Preferences`，在`Connection`中需要配置如下两个参数：
```
Oracle Home：c:instantclient
OCI Library：c:instantclientoci.dll
```
![](http://7xssec.com2.z0.glb.clouddn.com/tools-plsql-instantclient-1.png)
至此配置完成,现在就可以正常使用pl/sql developer了