---
title: Docker-搭建Discourse并整合到LNMP中
tags:
  - docker
categories:
  - Notes
date: 2015-12-10 11:27:06
---

## 准备工作

本人使用的是Hostus,10美元/年,洛杉矶节点。 

购买地址：[Hostus](https://my.hostus.us/aff.php?aff=822&pid=103)

默认系统为：Centos 6 64bit

## 创建Swap分区

使用`dd`命令创建一个`swap分区`

``` {bash}
dd if=/dev/zero of=/home/swap bs=1024 count=1048576
```

count的计算公式： `count=SIZE*1024 (size以MB为单位）`

这样就建立一个`/home/swap`的分区文件，大小为1G，接着需要格式化新建的SWAP分区：

```{bash}
mkswap /home/swap
```
再用`swapon`命令把这个文件分区变成swap分区

``` bash
swapon /home/swap
```
（关闭SWAP分区命令为：`swapoff /home/swap`）

再用`free -m`查看一下，可以看出swap扩大了。

为了能够让swap自动挂载，要修改`/etc/fstab`文件

```linux
vi /etc/fstab
```
在文件末尾加上
```
/home/swap swap swap default 0 0
```
这样就算重启系统，swap分区就不用手动挂载了。
## 安装Docker

```linux
yum install docker
sudo service docker start
sudo chkconfig docker on
```
## 安装Discourse
创建 `/var/discourse` 文件夹，克隆官方 Discourse Docker 镜像至其中，然后在拷贝一个配置文件，并命名为 `app.yml`:

```linux
mkdir /var/discourse
```
```linux
git clone https://github.com/discourse/discourse_docker.git /var/discourse
```
```linux
cd /var/discourse
```
```linux
cp samples/standalone.yml containers/app.yml
```
初始化：
```linux
./launcher bootstrap app
```
进入容器
```linux
./launcher enter app
```
输入：
```linux
rails console
```
```
SiteSetting.notification_email = 'admin@jourk.com'
```
## 在容器外安装 nginx
首先，确定容器没有工作：
```linux
cd /var/discourse
```
```linux
./launcher stop app
```
改变你的 `app.yml`，像这样：
```
- "templates/web.socketed.template.yml" # <-- 增加了

expose:

- "2222:22"
```
修改nginx配置文件
```xml
server {
listen 80;
# change this
server_name forum.riking.org;
location / {
   proxy_pass http://unix:/var/discourse/shared/standalone/nginx.http.sock:;
   proxy_set_header Host $http_host;
   proxy_http_version 1.1;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}
```

## 问题归总

问题1：mysql 启动错误-server PID file could not be found
```
# service mysqld stop
MySQL manager or server PID file could not be found! [FAILED]
```
解决办法：

首先查看一下进程
```
# ps aux |grep mysq*
root 10274 0.0 0.0 68160 1336 ? S 13:43 0:00 /bin/sh /usr/bin/mysqld_safe --datadir=/var/lib/mysql --pid-file=/var/lib/mysql/irxpert-test.pid

mysql 10353 0.0 1.0 344360 39464 ? Sl 13:43 0:00 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --user=mysql --log-error=/var/lib/mysql/irxpert-test.err --pid-file=/var/lib/mysql/irxpert-test.pid

root 11884 0.0 0.0 63384 760 pts/1 S+ 15:44 0:00 grep mysq*
```
如果看到上面的内容，那说明，Mysql的进程卡死了，这时用就要把这些卡死的进程都关闭
```
# kill 10274

# kill 10353
```

启动Mysql就ok了
```
# service mysql start
Starting MySQL. [ OK ]
```