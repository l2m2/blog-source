---
title: 离线安装Docker及镜像
toc: false
date: 2020-01-09 08:44:43
description: Docker傻逼日记
tags:
- Docker
---



以下描述均在CentOS 7环境测试。

## 离线安装docker

从Docker官网下载Docker的二进制文件。

官网下载地址：https://download.docker.com/linux/static/stable/x86_64/

解压

```
$ tar xvzf docker-20.10.6.tgz
```

移动到/usr/bin

```
$ mv docker/* /usr/bin/
```

制作服务文件docker.service

```
$ vim /etc/systemd/system/docker.service
[root@iZbp13sno1lc2yxlhjc4b3Z ~]#vim /etc/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
 
[Install]
WantedBy=multi-user.target
```

启动服务

```
$ chmod +x /etc/systemd/system/docker.service
$ systemctl daemon-reload
$ systemctl start docker
$ systemctl enable docker
```

如果`systemctl start docker`一直卡着不动，可以尝试先关闭防火墙再启动。启动之后再开启防火墙。

```
$ systemctl stop firewalld
$ systemctl start docker
$ systemctl start firewalld
```

## 离线安装docker镜像

以安装mssql为例：

在有网络的环境下pull

```bash
$ docker pull mcr.microsoft.com/mssql/server:2017-latest
```

然后保存为.tar文件

```bash
$ docker save --output mssql-docker-image.tar mcr.microsoft.com/mssql/server
$ ls
mssql-docker-image.tar
```

将此镜像文件copy到没有网络的环境下，然后load

```bash
$ docker load --input mssql-docker-image.tar 
```

看看是否load成功

```bash
$ docker image ls
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
docker.io/postgres               10.11               8007341bd20e        11 days ago         250 MB
mcr.microsoft.com/mssql/server   2017-latest         a4b86c4000e7        5 weeks ago         1.4 GB
```

## Reference

-  https://stackoverflow.com/questions/48125169/how-run-docker-images-without-connect-to-internet 

-  https://hub.docker.com/_/microsoft-mssql-server 