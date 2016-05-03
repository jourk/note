---
title: Windows-查看端口占用情况
tags:
  - windows
categories:
  - System
date: 2014-07-10 11:27:06
---
`开始–运行–cmd`进入命令提示符，输入`netstat -ano`即可看到所有连接的PID 之后在任务管理器中找到这个`PID`所对应的程序如果任务管理器中没有`PID`这一项，可以在任务管理器中选`查看`-`选择列`

> 假如我们需要确定谁占用了我们的`9050`端口

## 查看端口占用情况

打开windows命令行窗口

### 查看所有的端口占用情况

```
C:>netstat -ano

协议 本地地址 外部地址 状态 PID
TCP 127.0.0.1:1434 0.0.0.0:0 LISTENING 3236
TCP 127.0.0.1:5679 0.0.0.0:0 LISTENING 4168
TCP 127.0.0.1:7438 0.0.0.0:0 LISTENING 4168
TCP 127.0.0.1:8015 0.0.0.0:0 LISTENING 1456
TCP 192.168.3.230:139 0.0.0.0:0 LISTENING 4
TCP 192.168.3.230:1957 220.181.31.225:443 ESTABLISHED 3068
TCP 192.168.3.230:2020 183.62.96.189:1522 ESTABLISHED 1456
TCP 192.168.3.230:2927 117.79.91.18:80 ESTABLISHED 4732
TCP 192.168.3.230:2929 117.79.91.18:80 ESTABLISHED 4732
TCP 192.168.3.230:2930 117.79.91.18:80 ESTABLISHED 4732
TCP 192.168.3.230:2931 117.79.91.18:80 ESTABLISHED 4732
```

### 查看指定端口的占用情况

```
C:>netstat -aon|findstr “9050”

协议 本地地址 外部地址 状态 PID
TCP 127.0.0.1:9050 0.0.0.0:0 LISTENING 2016
```

P: 看到了吗，端口被进程号为`2016`的进程占用，继续执行下面命令： (也可以去任务管理器中查看pid对应的进程)

### 查看PID对应的进程

```
C:>tasklist|findstr “2016”

映像名称 PID 会话名 会话# 内存使用
========================= ======== ================
tor.exe 2016 Console 0 16,064 K
```

P:很清楚吧，`tor`占用了你的端口。

### 结束该进程

```
C:>taskkill /f /t /im tor.exe
```

## 通过PID查看对应的端口号

```
netstat -ano | findstr pid
netstat -ano | findstr 616
```
