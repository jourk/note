---
title: Linux-CentOS-虚拟机克隆后网卡配置问题
tags:
  - linux
categories:
  - Notes
date: 2014-07-28 11:27:06
---

>CentOS虚拟机克隆，物理地址会冲突，于是自动新建了网卡eth1，无法启动网卡。

## 解决办法
### 方式1

1、修改`/etc/sysconfig/network-scripts/ifcfg-eth0` 为 `/etc/sysconfig/network-scripts/ifcfg-eth1`

2、配置`/etc/sysconfig/network-scripts/ifcfg-eth1`

```linux-config
# 修改为eth1 
DEVICE="eth1" 
BOOTPROTO="static" 
# 硬件地址取自eth1（通过“ifconfig -a”获取）
HWADDR="00:0C:29:91:42:2C" 
MTU="1500" 
NM_CONTROLLED="yes" 
ONBOOT="yes" 
IPADDR=192.168.152.101 
NETMASK=255.255.255.0 
GATEWAY=192.168.152.2 
```

3、service network restart 

### 方式2

1、修改`/etc/udev/rules.d/70-persistent-net.rules`

拷贝eth1的硬件地址到eth0

删除eth1信息

2、配置`/etc/sysconfig/network-scripts/ifcfg-eth0`

``` linux-config
DEVICE="eth0" 
BOOTPROTO="static" 
HWADDR="00:0C:29:91:42:2C" 
MTU="1500" 
NM_CONTROLLED="yes" 
ONBOOT="yes" 
IPADDR=192.168.152.101 
NETMASK=255.255.255.0 
GATEWAY=192.168.152.2 
```

3、reboot