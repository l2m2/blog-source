---
title: 为正在运行的Docker容器添加Volume
toc: false
date: 2020-12-19 15:41:00
description: ...
tags:
- Docker
- PosetgreSQL
---

之前为客户安装PostgreSQL时忘记挂载Volume了，现在数据库越来越大，需要给正在运行的容器挂载Volume。

测试环境

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
```

先安装一个PostgreSQL 10，并不挂载Volume。

```bash
$ docker pull postgres:10
$ docker run --name pg10 -e POSTGRES_PASSWORD=123456 -p 5432:5432 -d postgres:10
```

检查下是否跑起来了。

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ec072f8d5031        postgres:10         "docker-entrypoint..."   14 seconds ago      Up 13 seconds       0.0.0.0:5432->5432/tcp   pg10
```

为了测试，创建一个数据库TEST, 创建表t1。

```sql
SELECT id, name FROM t1
-- 1	3
-- 2	4
```

**方案一**： 将容器中的数据拷贝出来，重新run一个容器。

```bash
$ docker cp pg10:/var/lib/postgresql/data/ /home/data/docker_data/pgdata/
$ docker container stop pg10
$ docker run --name pg10_new -e POSTGRES_PASSWORD=123456 -p 5432:5432 -d \
> -v /home/data/docker_data/pgdata/:/var/lib/postgresql/data postgres:10
```

确认无误后，删除之前的容器，再重命名新容器为原来的名称。

```bash
$ docker container rm pg10
$ docker rename pg10_new pg10
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
c38477f25557        postgres:10         "docker-entrypoint..."   2 minutes ago       Up 2 minutes        0.0.0.0:5432->5432/tcp   pg10
```

~~**方案二**：基于当前的容器生成新的镜像，基于新的镜像重新run一个容器。~~ 

此方案不适用于PostgreSQL的/var/lib/postgresql/data的挂载。

```bash
$ docker commit ec072f8d5031 pg10_new
sha256:457d9fde9e6d496ab0a919dd88536d66f49d784ff02509a66d7caf87fc66bf58
$ docker image ls
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
pg10_new                      latest              457d9fde9e6d        16 seconds ago      200 MB
docker.io/postgres            10                  8b85cd6eeb37        7 days ago          200 MB
$ docker container stop pg10
$ docker run --name pg10_new -e POSTGRES_PASSWORD=123456 -p 5432:5432 -d \
-v /home/data/docker_data/pgdata/:/var/lib/postgresql/data pg10_new
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ebfc992bde14        pg10_new            "docker-entrypoint..."   3 seconds ago       Up 2 seconds        0.0.0.0:5432->5432/tcp   pg10_new
```

## Reference

- https://stackoverflow.com/questions/28302178/how-can-i-add-a-volume-to-an-existing-docker-container
- https://docs.docker.com/storage/volumes/

