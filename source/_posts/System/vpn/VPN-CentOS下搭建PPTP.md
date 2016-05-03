---
title: VPN-CentOS下搭建PPTP
tags:
  - vpn
categories:
  - System
date: 2015-10-14 11:27:06
---

## 检测PPP和tun是否开启

命令如下

``` bash
# cat /dev/ppp
```

开启成功的标志：`cat: /dev/ppp: No such file or directory`或者 `cat: /dev/ppp: No such device or address`，可以继续安装过程.

开启不成功的标志：`cat: /dev/ppp: Permission denied`，请联系服务商开启。

``` bash
cat /dev/net/tun
```

显示结果为下面的文本，表明通过：

```
cat: /dev/net/tun: File descriptor in bad state
```

上述两条`只需一条通过`，即可安装`pptp`。如果还有其它问题，或者请你的服务商来解决这个问题。

## 安装ppp

``` bash
rpm -ivh http://poptop.sourceforge.net/yum/stable/rhel6/x86_64/ppp-2.4.5-33.0.rhel6.x86_64.rpm
```

## 安装pptp

``` bash
rpm -ivh https://www.dropbox.com/s/n5c9uekdrgyqtll/pptpd-1.4.0-1.el6.i686.rpm
```

## 配置pptp

首先我们要编辑`/etc/pptpd.conf`文件

``` bash
vi /etc/pptpd.conf
```

把下面字段前面的#去掉即可：

``` bash
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
```

接下来再编辑`/etc/ppp/options.pptpd`

``` bash
vi /etc/ppp/options.pptpd
```

去掉`ms-dns`前面的`#`，并修改成如下字段

``` bash
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

## 设置pptp VPN账号密码

我们需要编辑`/etc/ppp/chap-secrets`这个文件：

``` bash
vi /etc/ppp/chap-secrets
```

第一和第三个单词分别是用户和密码，注意都是`区分大小写`的：（此处搞错会导致651错误）

```
crazy pptpd crazymima *
```

## 修改内核设置，使其支持转发

编辑`/etc/sysctl.conf`文件：

将`net.ipv4.ip_forward`改为`1`：

```
net.ipv4.ip_forward=1
```

同时在`net.ipv4.tcp_syncookies = 1`前面加`#` ：

```
# net.ipv4.tcp_syncookies = 1
```

## 保存退出，并执行下面的命令来生效它

``` bash
sysctl -p
```

## 错误及解决办法

一般这是因为我们没有加载`bridge模块`，需要手工加载。解决过程如下：

``` bash
modprobe bridge
lsmod|grep bridge
```

但是openvz架构的vps执行的时候可能会出现新的错误.

这个问题是因为openvz的centos的模版的问题，要进行修复操作,修复也很简单,总共四个命令~

``` bash
rm -f /sbin/modprobe
ln -s /bin/true /sbin/modprobe
rm -f /sbin/sysctl
ln -s /bin/true /sbin/sysctl
```

这样就重建这两个模块的软连接,openvz的centos的模版就修复好了。

执行`sysctl -p`也果断没有报错了。

## 添加iptables转发规则

``` bash
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT –to-source 107.161.25.176
(OpenVZ, 107.161.25.176为你的VPS的公网IP地址)
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
(XEN)
```

## 保存iptables转发规则

``` bash
/etc/init.d/iptables save
```

iptables转发规则写错了会出现错误`678提示`（亲历），可用`iptables -F`删除旧规则再配置！

## 重启iptables

``` bash
/etc/init.d/iptables restart
```

## 重启pptp服务

``` bash
/etc/init.d/pptpd restart
```

## 设置开机自动运行服务

``` bash
chkconfig pptpd on
chkconfig iptables on
```

如果出现`错误619`则输入命令

``` bash
rm /dev/ppp
mknod /dev/ppp c 108 0
```

还不管用的话（有时出现`错误651`），请下载vps上`/var/log/messages`查看日志

## 在Window使用VPN

- 打开控制面板的`网络和共享中心`或者按`Win+X+P`
- 点击`连接到工作区`
- 创建新连接
- 选择`使用我的internet连接到VPN`

然后输入`VPN服务器地址`点确定就完事儿了。

VPN连接时输入自己VPN的用户名和密码。