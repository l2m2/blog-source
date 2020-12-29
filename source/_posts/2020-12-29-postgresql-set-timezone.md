---
title: PostgreSQL修改时区
toc: false
date: 2020-12-29 10:52:00
description: 修改当前session的时区，以及永久修改数据库服务的时区。
tags:
- PostgreSQL
---

**查询当前时区** 

```sql
SHOW TIME ZONE
-- Asia/Chongqing
```
**修改当前Session的时区** 
```sql
SET TIME ZONE 'Asia/Chongqing'
-- 此修改只会在当前Session生效
```
**永久修改数据库服务的时区** 

1. 找到postgresql.conf，通常是在PostgreSQL数据目录的根目录下。
- Windows下数据目录位置示例： `C:\Program Files\PostgreSQL\10\data` ，如果在安装过程中指定了数据目录，则在你指定的目录下。
- Linux下若是包安装的方式安装的，可使用下面的方式查看位置。
```bash
$ psql -U postgres -c 'SHOW config_file'
```

- Linux下若是Docker安装的，且挂载了数据目录的，则在挂载的数据目录下。
- Linux下若是Docker安装的，但没有挂载数据目录的，则需要进入Docker容器，在容器的 `/var/lib/posrgresql/data` 下
```bash
$ docker exec -it pg10 bash
root@d481d2896630:/# cd /var/lib/postgresql/data/
```

2. 找到postgresql.conf, 找到timezone的配置，修改它。
```
timezone = 'Asia/Chongqing'
```

3. 修改后重启数据库服务。

**测试** 
```sql
-- 查询当前时间
SELECT now()
```
