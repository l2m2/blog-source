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
$ yum install docker
```

检查版本

```bash
$ docker version
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: 
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

启动后台服务

```bash
$ systemctl start docker
$ systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-01-07 15:55:08 CST; 5s ago
```

设置开机自启动

```bash
$ systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

## 改变docker存储位置

通常我们系统所在的硬盘比较小，会再挂载一个容量更大的硬盘作为数据存储用。

以下命令用作改变docker默认的存储位置。

```bash
$ systemctl stop docker
$ mv /var/lib/docker/ /data/docker/
$ ln -s /data/docker/ /var/lib/docker
$ systemctl start docker
```

## 修改镜像源

```bash
$ cat /etc/docker/daemon.json 
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com"
  ]
}
```

## Docker安装PostgreSQL 10

```bash
$ docker pull postgres:10.11
$ docker run \
  --name "pg10" \
  -e POSTGRES_PASSWORD=123456 \
  -p 5432:5432 \
  -d --restart "always" \
  -v /home/data/pgdata:/var/lib/postgresql/data \
  postgres:10.11
```

--name 容器名称

-e 设置环境变量

-p 端口映射（前面的是host端口，后面的是container的端口）

-d 在后台运行容器

--restart 重启策略（ https://docs.docker.com/config/containers/start-containers-automatically/ ）

-v  将host机器的目录mount到container, 前面的是host机器目录，后面的是container的目录

测试容器是否运行成功

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
cdcc4e0769f5        postgres:10.11      "docker-entrypoint..."   35 seconds ago      Up 34 seconds       0.0.0.0:5432->5432/tcp   pg10
```

## Docker安装Microsoft SQL Server

```bash
$ docker pull mcr.microsoft.com/mssql/server:2017-latest
$ docker run \
  --name "mssql" \
  -e 'ACCEPT_EULA=Y' \
  -e 'SA_PASSWORD=123456' \
  -p 1433:1433 \
  -d --restart "always" \
  -v /home/data/mssqldata:/var/opt/mssql/data \
  mcr.microsoft.com/mssql/server:2017-latest
```

## Docker安装nginx

```bash
$ docker run --name topikm6doc \
-p 8888:80 \
-d \ 
--rm \
-v /data/docker_data/topikm6doc/html:/usr/share/nginx/html \
nginx
```

## 进入容器

```bash
$ docker exec -it pg10 bash
```

退出

```bash
$ exit
```

## 常用docker命令

```bash
# 从docker容器拷贝文件到宿主机
# 拷贝mysql容器内的mysqld.cnf到当前目录
$ docker cp mysql:/etc/mysql/mysql.conf.d/mysqld.cnf .
# 从宿主机拷贝文件到docker容器
# 拷贝当前目录的mysqld.cnf到mysql容器的mysql.conf.d目录
$ docker cp mysqld.cnf mysql:/etc/mysql/mysql.conf.d
# 查看已挂载的目录
$ docker inspect topikm6doc | grep Mounts -A 20
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/docker_data/topikm6doc/html",
                "Destination": "/usr/share/nginx/html",
                "Mode": "",
                "RW": true,
                "Propagation": "rslave"
            }
        ],
# 查看所有Volume
$ docker volume ls
# 查看某个具体Volume的信息
$ docker volume inspect thdash_app-db-data
```



## Reference

-  https://stackoverflow.com/questions/24309526/how-to-change-the-docker-image-installation-directory 