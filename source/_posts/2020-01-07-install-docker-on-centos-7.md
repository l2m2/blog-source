---
title: CentOS 7 安装Docker
toc: false
date: 2020-01-07 15:14:11
description: 记录一下
tags:
- Docker
- CentOS
---

## 安装Docker

yum安装

```bash
[root@localhost ~]# yum install docker
```

检查版本

```bash
[root@localhost ~]# docker version
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: 
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

启动后台服务

```bash
[root@localhost ~]# systemctl start docker
[root@localhost ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-01-07 15:55:08 CST; 5s ago
```

设置开机自启动

```bash
[root@localhost ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

## 改变docker存储位置

通常我们系统所在的硬盘比较小，会再挂载一个容量更大的硬盘作为数据存储用。

以下命令用作改变docker默认的存储位置。

```bash
[root@localhost ~]# systemctl stop docker
[root@localhost ~]# mv /var/lib/docker/ /data/docker/
[root@localhost ~]# ln -s /data/docker/ /var/lib/docker
[root@localhost ~]# systemctl start docker
```

## Docker安装PostgreSQL 10

```bash
[root@localhost ~]# docker pull postgres:10.11
[root@localhost home]# docker run --name "pg10" -e POSTGRES_PASSWORD=TopLinker0510 -p 5432:5432 -d --restart "unless-stopped" -v /home/data/pgdata:/var/lib/postgresql/data postgres:10.11
```

--name 容器名称

-e 设置环境变量

-p 端口映射（前面的是host端口，后面的是container的端口）

-d 在后台运行容器

--restart 重启策略（ https://docs.docker.com/config/containers/start-containers-automatically/ ）

-v  将host机器的目录mount到container, 前面的是host机器目录，后面的是container的目录

测试容器是否运行成功

```bash
[root@localhost home]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
cdcc4e0769f5        postgres:10.11      "docker-entrypoint..."   35 seconds ago      Up 34 seconds       0.0.0.0:5432->5432/tcp   pg10
```

## Reference

-  https://stackoverflow.com/questions/24309526/how-to-change-the-docker-image-installation-directory 