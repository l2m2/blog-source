---
title: PostgreSQL创建只读用户
toc: false
date: 2021-01-04 17:26:00
description: ...
tags:
- PostgreSQL
---

## 创建只读用户

以超级用户登录，连接到TOPDFM_DEV_V6数据库。

```sql
-- 创建
CREATE ROLE reporter WITH LOGIN PASSWORD '123456' 
NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION VALID UNTIL 'infinity';

-- 授权
GRANT CONNECT ON DATABASE "TOPDFM_DEV_V6" TO reporter;
GRANT USAGE ON SCHEMA public TO reporter;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO reporter;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO reporter;

-- 为将来的表授权
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO reporter;
```

## 删除只读用户

以超级用户登录，连接到postgres数据库。

```sql
REASSIGN OWNED BY reporter TO postgres;
DROP OWNED BY reporter;
DROP ROLE reporter;
```

## Reference

- https://stackoverflow.com/questions/760210/how-do-you-create-a-read-only-user-in-postgresql

- https://dba.stackexchange.com/questions/155332/find-objects-linked-to-a-postgresql-role