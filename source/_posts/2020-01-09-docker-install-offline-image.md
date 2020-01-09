---
title: 离线安装Docker镜像
toc: false
date: 2020-01-09 08:44:43
description: Docker傻逼日记
tags:
- Docker
---

以安装mssql为例：

在有网络的环境下pull

```bash
[root@192 home]# docker pull mcr.microsoft.com/mssql/server:2017-latest
```

然后保存为.tar文件

```bash
[root@iZuf61ti9rpeooj054hbpgZ dirty]# docker save --output mssql-docker-image.tar mcr.microsoft.com/mssql/server 
[root@iZuf61ti9rpeooj054hbpgZ dirty]# ls
mssql-docker-image.tar
```

将此镜像文件copy到没有网络的环境下，然后load

```bash
[root@localhost ~]#docker load --input mssql-docker-image.tar
```

## Reference

-  https://stackoverflow.com/questions/48125169/how-run-docker-images-without-connect-to-internet 