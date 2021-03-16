---
title: 通过端口号查找对应的进程信息
toc: false
date: 2021-03-12 13:49:00
description: ...
tags:
- Linux
---

1. 先通过netstat找到对应的PID

```shell
$ netstat -tulpn | grep 8083
tcp6       0      0 :::8083                 :::*                    LISTEN      155343/java
```

对应的PID是155343

netstat的参数解释

```
-t, --tcp 仅显示tcp相关选项
-u, --udp 仅显示udp相关选项
-l, --listening 仅显示侦听套接字
-p, --program 显示每个套接字要连接到的程序的PID和名称
-n, --numeric 显示数字地址而不是尝试确定符号,主机，端口或用户名。
```

2. 查找这个进程是哪个exe在运行

```shell
$ ls -l /proc/155343/exe
lrwxrwxrwx 1 root root 0 Aug  1  2020 /proc/155343/exe -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/jre/bin/java
```

3. 查找这个进程当前的工作目录

```shell
$ ls -l /proc/155343/cwd
lrwxrwxrwx 1 root root 0 Mar 16 16:55 /proc/155343/cwd -> /home/data/toplinker/topibd-http-server-1.6.15/bin
```

/proc/pid下各子目录的含义

```
/cmdline: 该文件保存了进程的完整命令行
/cwd: 指向进程当前的工作目录
/environ: 该文件保存进程的环境变量
/exe: 指向被执行的二进制文件
/fd: 进程所打开的每个文件都有一个符号连接在该子目录里, 以文件描述符命名, 这个名字实际上是指向真正的文件的符号连接
```



## Reference

- https://man7.org/linux/man-pages/man8/netstat.8.html