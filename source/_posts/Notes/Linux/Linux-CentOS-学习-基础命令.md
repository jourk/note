---
title: Linux-CentOS-学习-基础命令
tags:
  - linux
categories:
  - Notes
date: 2014-09-12 11:27:06
---
## 日期时间

|命令|说明|
|---|---|
|date|查看、设置当前系统时间|
|+%Y–%m–%d|格式化显示日期|
|date -u|显示UTC时间| 
|date -s|修改系统时间|
|hwclock（clock）|显示硬件时钟时间|
|cal|查看日历|
|uptime|查看系统运行时间|

## 输出、查看命令

|命令|说明|
|---|---|
|echo|显示输入的内容|
|cat|显示文件内容|
|head|显示文件的头几行(默认10行)|
|  -n |指定显示的行数|
|tail|显示文件的末尾几行(默认10行)
|  -n| 指定显示的行数|
|  -f| 追踪显示文件更新(一般用于查看日志,命令不会退出,而是持续显示新加入的内容)|
|more|用于翻页显示文件内容(只能向下翻)|
|less|用于翻页显示文件内容(可上下翻)|

## 查看硬件信息

|命令|说明|
|---|---|
|lspci|查看PCI设备|
|  -v |查看详细信息|
|lsusb|查看USB设备|
|  -v |查看详细信息|
|lsmod|查看加载的模块(驱动)|

4、关机、重启

`shutdown`命令可以关闭、重启计算机。
  
shutdown [-h|-r] 时间

  -h 关闭计算机
  -r 重新启动

如：

|命令|说明|
|---|---|
|shutdown -h now| 立即关机|
|shutdown -h +10| 10分钟后关机|
|shutdown -h 23:30| 晚上十一点半关机|
|shutdown -r now| 立即重启|
|poweroff|立即关闭计算机|
|reboot|立即重启计算机|

## 归档、压缩

### zip命令可以压缩文件。

```
zip filename.zip file1 fiel2 …
```

|命令|说明|Example|
|---|---|---|
|unzip|解压缩zip文件|unzip filename.zip|
|gzip|压缩文件|gzip filename|

### tar命令可以归档文件

```
tar -cvf out.tar file1 file2 …

tar -xvf filename.tar

tar -cvzf out.tar.gz file file1 …
```

- -z 参数将归档后的归档文件进行gzip压缩以减少大小

## 查找

`locate`命令快速查找文件、文件夹

```
locate keyword
```

此命令需预先建立数据库，数据库默认每天更新一次，可以使用”updatedb“命令手工建立、更新数据库

### find命令可以高级查找文件、文件夹

|命令|说明|
|---|---|
|find . -name *keyword*| 在当前目录查找文件名中包含”keyword”的文件|
|find / -name *.conf |在根目录中查找文件名以”.conf”结尾的文件|
|find / -perm 777| 在根目录中查找权限为”777″的文件|
|find / -type d| 在根目录中查找类型为”d”(目录)的文件|
|find . -name “a*” -exec ls -l {} |立即重启|

## FIND查找条件

`find`支持很多种查找条件,常用的如下：

|属性|说明|
|---|---|
|-name| 按名称查找|
|-perm| 按权限查找|
|-user| 按所有者查找|
|-group| 按所属用户组查找|
|-ctime| 按创建时间查找|
|-type| 按文件类型查找|
|-size| 按文件大小查找|