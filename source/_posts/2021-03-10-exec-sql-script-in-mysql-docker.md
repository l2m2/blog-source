---
title: 在MySQL Docker中执行SQL文件
toc: false
date: 2021-03-10 15:42:00
description: ...
tags:
- Docker
- MySQL
---

将sql脚本拷贝到docker容器内

```shell
$ docker cp test.sql mysql5.7:/root/
```

进入容器

```shell
$ docker exec -it mysql5.7 bash
```

执行mysql，-u 指定用户名-D 指定数据库

```shell
$ mysql -u root -D codepush -p 
```

执行脚本

```shell
> \. test.sql
```

