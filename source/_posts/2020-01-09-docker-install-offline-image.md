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

在有docker环境的机器上运行：

```bash
$ yumdownloader --resolve docker-ce
```

将会生成`docker-ce-19.03.11-3.el7.x86_64.rpm`类似的rpm包。

再在目标机上安装rpm包。

```bash
$ rpm -ivh docker-ce-19.03.11-3.el7.x86_64.rpm
```

安装过程中可能会缺少其他rpm包，再去下载对应的离线rpm包安装即可。

安装完成后启动docker。

```bash
$ systemctl start docker
$ systemctl enable docker
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