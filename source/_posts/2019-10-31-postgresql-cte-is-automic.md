---
title: PostgreSQL - CTE的原子性
toc: false
date: 2019-10-31 15:53:31
description: 你可能不知道的SQL
tags:
- PostgreSQL
- SQL
---

## CTE (Common Table Expressions)

**什么是CTE? **

 A common table expression (CTE) can be thought of as a temporary result set that is defined within the execution scope of a single SELECT, INSERT, UPDATE, DELETE, or CREATE VIEW statement. A CTE is similar to a derived table in that it is not stored as an object and lasts only for the duration of the query. Unlike a derived table, a CTE can be self-referencing and can be referenced multiple times in the same query. 

可以将公用表表达式（CTE）视为在单个SELECT，INSERT，UPDATE，DELETE或CREATE VIEW语句的执行范围内定义的临时结果集。 CTE与派生表相似，因为它不存储为对象，并且仅在查询期间持续存在。 与派生表不同，CTE可以是自引用的，并且可以在同一查询中多次引用。

**它与临时表的区别**

临时表是实实在在存在的表，而CTE不是。

**什么时候用得到**

- 创建递归查询。
- 当不需要通用视图时，可以用它代替视图； 也就是说，您不必将定义存储在元数据中。
- 在一个复杂的SQL中中多次引用同一个子查询。

## CTE IS AUTOMIC

我们用一个例子来证明CTE是原子性的。

有两张表, 一张主表 p_table, 一张次表 s_table。

```plsql
CREATE TABLE P_table(
	id INT PRIMARY KEY,
	content TEXT
);
CREATE TABLE s_table(
	id serial PRIMARY KEY,
	rid INT,
	content TEXT
);
```

先往次表插入数据，再根据次表数据的插入结果往主表插入数据。

```plsql
WITH s AS (
	INSERT INTO s_table(rid, content)
	VALUES (1, 'secondary data 1')
	RETURNING rid
), p AS (
	INSERT INTO p_table (id, content)
	SELECT rid, 'primary data 1 after secondary data 1 inserted.'
	FROM s
	RETURNING id
)
SELECT id FROM p
```

执行结果：

p_stable

| id   | content                                         |
| ---- | ----------------------------------------------- |
| 1    | primary data 1 after secondary data 1 inserted. |

s_table

| id   | rid  | content          |
| ---- | ---- | ---------------- |
| 1    | 1    | secondary data 1 |

再来看一个例子。

```plsql
WITH s AS (
	INSERT INTO s_table (rid, content)
	VALUES (1, 'secondary data 1 shouldn''t been inserted')
	RETURNING rid
), p AS (
	INSERT INTO p_table (id, content)
	SELECT rid, 'primary data 1 after secondary data inserted but shouldn''t been insert'
	FROM s
	RETURNING id
)
SELECT id FROM p
```

往次表插入数据，然后rid为1，然后再往主表插入id为1的数据。

执行结果：

> ERROR:  duplicate key value violates unique constraint "p_table_pkey"
> DETAIL:  Key (id)=(1) already exists. 

两张表的数据都没有变化。

**由此可见，CTE是原子性的。**

## 参考

- [你可能不知道的SQL - 刘鑫](https://qcon.infoq.cn/2019/shanghai/presentation/1985)
-  https://stackoverflow.com/questions/4740748/when-to-use-common-table-expression-cte 