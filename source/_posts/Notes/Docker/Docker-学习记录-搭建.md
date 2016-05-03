---
title: Docker-学习记录-搭建
tags:
  - mysql
categories:
  - Notes
date: 2015-10-29 11:27:06
---
## 一、环境:
- 服务器:DigitalOcean [点击申请](https://www.digitalocean.com/?refcode=edccd86e640b)
- 操作系统:CentOS 7

## 二、安装前准备工作
Docker必须安装在64-bit操作系统上，CentOS 7的内核版本不低于`3.10`。

使用`uname –r`检查内核版本:
```bash
$ uname –r
```

`3.10.0-229.el7.x86_64`

## 三、安装

有两种方式安装Docker。你可以使用yum或者可以使用curl从get.docker.com网站下载。
### 通过yum安装

1、使用root用户登录

2、确保系统使用最新的软件包

```bash
$ sudo yum update
```

3、添加yum repo

```bash
$ cat >/etc/yum.repos.d/docker.repo <<-EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```

4、安装Docker包

```bash
$ sudo yum install docker-engine
```

5、打开Docker进程

```bash
$ sudo service docker start
```

6、运行测试确保docker安装完成

```bash
$ sudo docker run hello-world
```
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from hello-world
a8219747be10: Pull complete
91c95931e552: Already exists
hello-world:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:aa03e5d0d5553b4c3473e89c8619cf79df368babd1.7.1cf5daeb82aab55838d
Status: Downloaded newer image for hello-world:latest
Hello from Docker.
This message shows that your installation appears to be working correctly.
To generate this message, Docker took the following steps:
1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
(Assuming it was not already locally available.)
3. The Docker daemon created a new container from that image which runs the
executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it
to your terminal.
To try something more ambitious, you can run an Ubuntu container with:
$ docker run -it ubuntu bash
For more examples and ideas, visit:
http://docs.docker.com/userguide/
```

### 使用脚本安装


1、用root用户登录


2、确保系统使用最新的软件包

```bash
$ sudo yum update
```

3、运行Docker安装脚本

```bash
$ curl -sSL https://get.docker.com/ | sh
```

这个脚本将添加 docker.repo 并安装Docker。

4、开启Docker进程

```bash
$ sudo service docker start
```

5、运行测试确保docker安装完成

```bash
$ sudo docker run hello-world
```

## 四、创建一个docker group
为了避免每次都要输入sudo命令，可以使用docker创建一个用户组docker，并添加用户。当docker进程启动，docker用户组的成员将对docker拥有读写权限。

创建docker组并添加用户


1、使用一个具有sudo权限的用户登录centos


2、创建docker组并添加用户

```bash
$ sudo usermod -aG docker your_username
```

3、注销之后登录

4、验证运行docker而不使用sudo

5、开机启动docker进程

启动计算机时即启动docker

```bash
$ sudo chkconfig docker on
```

## 五、卸载
可以使用yum卸载docker软件


1、列出所有安装的包

```bash
$ yum list installed | grep docker
docker-engine.x86_64 1.7.1-1.el7 @/docker-engine-1.7.1-1.el7.x86_64.rpm
```

2、移除包

```bash
$ sudo yum -y remove docker-engine.x86_64
```

这个命令不会移除主机上的镜像、容器、卷和用户创建的配置文件。

3、删除主机上的镜像、容器、卷

```bash
$ rm -rf /var/lib/docker
```

4、找到并删除用户创建的配置文件