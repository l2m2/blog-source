---
title: PostgreSQL中的LATERAL查询
toc: false
date: 2021-03-25 14:42:00
description: ...
tags:
- PostgreSQL
---

在我的实际场景中，有两张表，一张用户表sys_user。

| id   | username |
| ---- | -------- |
| 1    | Leon     |
| 2    | Dirty    |

还有一张用户与角色的关系表user_role_rel。

| user_id | role_id |
| ------- | ------- |
| 1       | 2       |
| 1       | 4       |
| 2       | 3       |

现在需要查询出用户信息，信息中包含角色的id列表。大概长这样。

| id   | username | roles |
| ---- | -------- | ----- |
| 1    | Leon     | {2,4} |
| 2    | Dirty    | {3}   |

普通的方式，我们可以先连接再聚合。

```sql
WITH t AS (
	SELECT A.id, array_agg(B.role_id) AS roles
	FROM sys_user A 
	LEFT JOIN sys_user_role_rel B ON A.id = B.user_id
	GROUP BY A.id
)
SELECT A.*, t.roles FROM sys_user A LEFT JOIN t ON A.id = t.id
```

**PostgreSQL给我们提供了另外一种方式，LATERAL子查询。**

```sql
SELECT A.*, C.roles 
FROM sys_user A 
LEFT JOIN LATERAL ( 
  SELECT ARRAY_AGG ( B.role_id ) AS roles 
	FROM sys_user_role_rel B 
	WHERE B.user_id = A.id
) C ON TRUE
```

LATERAL允许引用前面的FROM项提供的列。

## Reference

- http://www.postgres.cn/docs/10/queries-table-expressions.html#QUERIES-FROM
- https://stackoverflow.com/questions/58267279/postgresql-left-join-query-object-array-aggregate