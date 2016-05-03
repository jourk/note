---
title: Windows-更换密钥方法
tags:
  - windows
categories:
  - System
date: 2015-06-02 11:27:06
---
# Windows 更换密钥方法
## 修改和替换密钥的几种方法

### 更改密钥命令：

`Win+x`—命令提示符（以管理员身份）—输入：`slmgr /ipk` +序列号（要替换的密钥）–回车即可！或：记事本编写`slmgr /ipk +序列号`保存为.bat格式的批处理文件即可！

### 替换密钥命令：

`Win+x`—命令提示符（以管理员身份）—输入：`slmgr /upk`—回车即可！记事本编写`slmgr /upk`保存为.bat格式的批处理文件即可！

### 取消密钥命令：

`Win+x`—命令提示符（以管理员身份）—输入：`slmgr /ato`—回车即可！记事本编写`slmgr /ato`保存为.bat格式的批处理文件即可！

## 找到可以激活的密钥，替换一下即可

## 关于：“slmgr”命令的几种用法

1）查看许可证的概要信息（假设为当前许可证；且系统在C盘，下同），则可通过以下两种方式查看。

运行命令下

```
slmgr.vbs -dli
```

命令提示符命令下

```
cscripq C:\windows\system32\slmgr.vbs -dli
```

2）显示许可证激活状态的截止日期，也可通过以下两种方式查看。

   运行命令下：`slmgr.vbs -xpr`

   命令提示符命令下：`cscripq C:\windows\system32\slmgr.vbs -xpr`

3）查看许可证详细信息，也可通过以下两种方式查看。

   运行命令下：`slmgr.vbs -dlv`

   命令提示符命令下：`cscripq C:\windows\system32\slmgr.vbs -dlv`

4）导入OEM证书方法

   运行命令下：`slmgr.vbs -ilc D:123.XRM-MS`，后面为OEM证书完整路径

5）卸载当前产品密钥

   运行命令下：`slmgr.vbs -upk`，即可卸载当前产品密钥，重启计算机会出现输入密钥和联网激活界面。

# Office 更换密钥方法

## 基本操作

管理员身份运行CMD，然后按照以下步骤操作：

1)定位到Office2013（64位版本）的安装目录：

32位的操作系统：`CD “%PROGRAMFILES%\Microsoft Office\Office15”`

64位的操作系统：`CD “%PROGRAMFILES(x86)%\Microsoft Office\Office15”`

2)安装产品密钥（取代现有密钥）：

```
cscript ospp.vbs /inpkey:XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```

3)删除KEY，XXXXX为要删key的最后5位字母：

```
cscript ospp.vbs /unpkey:XXXXX
```

例1(安装产品密钥，适用于32位操作系统)：

```
cscript “%PROGRAMFILES%\Microsoft Office\Office15\ospp.vbs” /inpkey:XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```
例2(安装产品密钥，适用于64位操作系统)：

```
cscript “%PROGRAMFILES(x86)%\Microsoft Office\Office15\ospp.vbs” /inpkey:XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```

例3(删除产品密钥，适用于32位操作系统)：

```
cscript cscript “%PROGRAMFILES%\Microsoft Office\Office15\ospp.vbs” /unpkey:XXXXX
```

例4(删除产品密钥，适用于64位操作系统)：

```
cscript cscript “%PROGRAMFILES(x86)%\Microsoft Office\Office15\ospp.vbs” /unpkey:XXXXX
```

## ospp.vbs命令详解

ospp.vbs文件在哪里？

该脚本位于%PROGRAMFILES%\Microsoft Office\Office15文件夹中。如果您运行的是32位Office 2013的64位操作系统，该脚本位于%PROGRAMFILES(x86)%\Microsoft Office\Office15文件夹中。

ospp.vbs命令介绍

命令有很多，不一一全部介绍了，说说几个激活过程中的常用命令。为了让大家看得更明白，下面我以我虚拟机里的Office 2003 Pro Plus做个演示。

```
ospp.vbs
cscript ospp.vbs /dstatus
```
显示当前已安装产品密钥的许可证信息。可以查看到自已安裝的版本有多少个序列号。

```
ospp.vbs
cscript ospp.vbs /unpkey:xxxxx
```

卸载已安装的产品密钥。后面的数字是密钥的最后5位数。

此时再执行`cscript ospp.vbs /dstatus`发现产品密钥已经没有了，我重新进行导入。

```
ospp.vbs
cscript ospp.vbs /inpkey:xxxxx……
```

安装、替换现有的产品密钥。和上面的过程刚好相反。

```
cscript ospp.vbs /sethst:x.x.x.x
```

设置KMS主机名。一般为IP地址。

```
cscript ospp.vbs /act
```

激活当前安装的Office。

```
cscript ospp.vbs /remhst
```
删除KMS主机名。