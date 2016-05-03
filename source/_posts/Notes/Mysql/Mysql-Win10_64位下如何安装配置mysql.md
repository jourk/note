---
title: Mysql-Win10_64位下如何安装配置mysql
tags:
  - mysql
categories:
  - Notes
date: 2015-11-06 11:27:06
---

## Mysql.zip下载

[官方下载](http://www.mysql.com/downloads/)

## 解压到D:\Mysql

## 在D:\Mysql下新建my.ini配置文件

内容如下：
```ini
 [mysql]
 # 设置mysql客户端默认字符集
default-character-set=utf8 
 [mysqld]
 #设置3306端口
port = 3306 
 # 设置mysql的安装目录
basedir=D:/MySQL
 # 设置mysql数据库的数据的存放目录
datadir=D:/MySQL/data/
 # 允许最大连接数
max_connections=200
 # 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
 # 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB 
 sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```
重点是以下配置，其中datadir的目录名称必须是：`D:\Mysqldata`。

## 修改环境变量
在windows系统环境变量path，加入如下内容
```
D:\Mysql\bin;（注意加分号）
```
## 将mysql注册为windows系统服务

具体操作是在命令行中执行以下命令（需要以管理员身份运行命令行）：

需要切换到bin目录，否则，会将服务目录指定为C:\Program Files\MySQL\MySQL Server 5.7\mysqld

增加服务命令：
```linux
mysqld install MySQL --defaults-file="D:\MySQLmy.ini"
```
移除服务命令为：
```linux
mysqld remove
```
## 第5步成功后，打开系统服务管理

可以看到mysql系统服务

使用 `mysqld –console` 启动

可以显示出启动错误信息

在命令行启动mysql命令为：
```linux
net start mysql
```
关闭mysql命令为：
```linux
net stop mysql
```
## 修改root的密码为`jourk123`

命令行执行：`mysql –uroot`
```linux
mysql> show databases;
mysql> use mysql;
mysql> UPDATE user SET password=PASSWORD('jourk123') WHERE user='root';
mysql> FLUSH PRIVILEGES;
mysql> QUIT
```
## 远程登陆配置

允许root用户在任何地方进行远程登录，并具有所有库任何操作权限，具体操作如下：

1）在本机先使用root用户登录mysql：

命令行执行：`mysql -u root -p`

输入密码（第7步中设置的密码）：`jourk123`

2）进行授权操作：
```linux
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
```
重载授权表：
```linux
mysql> FLUSH PRIVILEGES;
```
退出`mysql：quit`