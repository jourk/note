---
title: Wordpress-总结-网站搭建篇
tags:
  - wordpress
categories:
  - Website
date: 2014-06-28 11:27:06
---

> 上篇介绍了主机、系统和博客平台，这篇就要详细写环境的搭建，以作备忘。

## 安装XFCE轻量级桌面

### 安装最快的镜像选择器

``` bash
yum -y install yum-fastestmirror
yum update
```

### 关闭NetworkManager开机启动

``` bash
chkconfig –list NetworkManager
```

停止开机自启

``` bash
chkconfig –level 123456 NetworkManager off
```

### 安装图形化界面的必要组件

``` bash
yum groupinstall “X Window System”
```

### 安装epel yum源

``` bash
wget http://download.Fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm
yum groupinstall XFCE
yum groupinstall Fonts （可选安装）
```

### 安装语言支持

``` bash
yum install fonts-chinese （CentOS 5 安装中文字体）
yum groupinstall chinese-support （CentOS 6 安装中文字体）
yum install nautilus-open-terminal （桌面右键菜单在终端中打开，需重启）
```

### 安装Firefox火狐浏览器

``` bash
yum -y install firefox
```

### 安装Firefox的flash player插件

** flash源 **

``` bash
#rpm -ivh http://linuxdownload.adobe.com/adobe-release/adobe-release-x86_64-1.0-1.noarch.rpm
#yum install flash-plugin
```

使用`firefox`访问`www.adobe.com`，点击`Get Flash Player`下载rpm包 在rpm文件所在文件夹右键，进入终端（Open Terminal Here），使用`rpm -ivh flash-plugin-*.rpm`进行安装。 在firefox的地址栏输入`about:plugins`查看是否安装成功~

### 重新启动VPS

``` bash
reboot
```

## 安装VNC server

### yum安装

``` bash
# yum install vnc vnc-server (适用CentOS 5)
# yum install tigervnc-server (适用CentOS 6)
```

### 修改vncservers配置

向`/etc/sysconfig/vncservers`里写入两行内容，懒人可以直接用如下命令写入：

``` bash
# echo ‘VNCSERVERS=”1:root”‘ >> /etc/sysconfig/vncservers
# echo ‘VNCSERVERARGS[1]=”-geometry 1024×650″‘ >> /etc/sysconfig/vncservers
```

### 启动VNC

首次启动，会要求输入两遍密码

``` bash
# vncserver
```

修改密码用此命令

``` bash
# vncpasswd
```

### 添加startxfce4

如果安装的是`Gnome`，把`~/.vnc/xstartup`最后一行`twm`替换为`gnome-session`，懒人请执行以下语句替换

``` bash
# sed -i ‘s/twm/gnome-session/g’ ~/.vnc/xstartup
```

如果安装的是`xfce`，则执行如下语句：

``` bash
# mv ~/.vnc/xstartup ~/.vnc/xstartup.bak
# echo ‘#!/bin/sh’ >> ~/.vnc/xstartup
# echo ‘/usr/bin/startxfce4‘ >> ~/.vnc/xstartup
```

### 给予权限，设置开机自启动等

``` bash
# chmod +x ~/.vnc/xstartup
# service vncserver restart
```

### 设置开机自启

``` bash
# chkconfig vncserver on
```

### 下载安装VNC View

本地安装VNC View，输入`ip:1`连接到远程桌面

### VNCServer使用方法

``` bash
# vncserver :1 启动:1
# vncserver :2 启动:2
# ps -ef|grep -i xvnc 查看已启动的server
# vncserver -kill :1 杀死:1
```

### 常见问题

** 问题一 **

``` bash
# service vncserver restart
Shutting down VNC server: 1:root [FAILED]
Starting VNC server: 1:root A VNC server is already running as :1
[FAILED]
```

** 故障原因 **：`/etc/hosts与/etc/sysconfig/network`文件中的`hostname`不一致。

一般改掉`/etc/hosts`中的`hostname`，再重启vncserver就好了。

** 问题二 **

``` bash
# vncserver
xauth: (stdin):1: bad display name “os1:4” in “add” command
```

** 故障原因 **：原因同上。

## 安装LNMP及网站

### LNMP及基本配置

#### 首先需要下载并执行LNMP一键安装包的执行脚本

CentOS系统下执行

``` bash
wget -c http://soft.vpser.net/lnmp/lnmp1.0-full.tar.gz && tar zxvf lnmp1.0-full.tar.gz && cd lnmp1.0-full && ./centos.sh
```

Debian系统下执行

``` bash
wget -c http://soft.vpser.net/lnmp/lnmp1.0-full.tar.gz && tar zxvf lnmp1.0-full.tar.gz && cd lnmp1.0-full && ./debian.sh
```

Ubuntu系统下执行

``` bash
wget -c http://soft.vpser.net/lnmp/lnmp1.0-full.tar.gz && tar zxvf lnmp1.0-full.tar.gz && cd lnmp1.0-full && ./ubuntu.sh
```

** 注意 **：安装过程中会依次要求您输入如下的相应信息：

- MySQL的root用户密码;
- 是否安装InnoDB的数据库引擎，请输入`y`;
- PHP版本的选择，请输入`y`;
- MySQL版本的选择，请输入`y`。

#### 配置Nginx文件上传最大值

通过`vi`编辑器编辑`/usr/local/nginx/conf/nginx.conf`文件，

``` bash
vi /usr/local/nginx/conf/nginx.conf
```

在`http{}`字段里面找到`client_max_body_size`,把后面的`50m`改成`1024m`,也就是 `client_max_body_size 1024m`.

#### 修改`php.ini`文件上传大小

``` bash
vi /usr/local/php/etc/php.ini
```

|参数|说明|
|---|---|
|file_uploads = on|是否允许通过HTTP上传文件的开关。默认为ON即是开|
|upload_tmp_dir|文件上传至服务器上存储临时文件的地方，如果没指定就会用系统默认的临时文件夹|
|upload_max_filesize = 8m|望文生意，即允许上传文件大小的最大值。默认为2M|
|post_max_size = 8m|指通过表单POST给PHP的所能接收的最大值，包括表单里的所有值。默认为8M|
|max_execution_time = 600|每个PHP页面运行的最大时间值(秒)，默认30秒|
|max_input_time = 600|每个PHP页面接收数据所需的最大时间，默认60秒|
|memory_limit = 8m|每个PHP页面所吃掉的最大内存，默认8M|

### 安装Wordpress

#### 创建Nginx的虚拟主机

执行脚本创建默认nginx脚本

``` bash
sh  /root/vhost.sh
```

这个过程中会要求你来输入一些相关信息。

- 要求输入域名：`blog.jourk.com`;
- 要求添加其他域名的时候，输入`n`;
- 要求输入默认存储目录的时候，按回车键，`Enter`;
- Allow Rewrite rule? 选择`y`;
- 输入`wordpress`建立伪静态
- Allow access_log？ 选择`y`。

接下去就是一路回车…

#### 修改`php.ini`删除`scandir`

1．打开`php.ini`文件

``` bash
#vi /usr/local/php/etc/php.ini
```

2．使用vi快捷功能中的查找功能查找scandir`:/scandir`

3．把第2步中查找到的`scandir`删除，然后保存文件，再重启服务器就可以了。

### 下载wordpress文件并解压到对应目录中

``` bash
cd /home/wwwroot/blog.jourk.com
wget http://cn.wordpress.org/wordpress-3.9-zh_CN.tar.gz
tar zxvf wordpress-3.9-zh_CN.tar.gz
```

### 将所有wordpress内的文件复制到网站根目录并赋予www用户权限

``` bash
cd wordpress
cp -R * /home/wwwroot/blog.jourk.com
cd /home/wwwroot
chown -Rf www:www blog.jourk.com
```

### 进入mysql并创建blog数据库

``` bash
mysql -u root -p
>show databases;
>create database blog;
```

### 重启lnmp

``` bash
/root/lnmp restart
```

### 进入`blog.jourk.com`进行安装并使用`UpdraftPlus`插件进行数据还原

至此wordpress安装完毕