---
title: PostgreSQL - 继承
toc: false
date: 2020-12-08 13:28:00
description: 通常我们不会在数据库设计时使用继承，本文只是了解下。
tags:
- PostgreSQL
---

本文所有示例基于PostgreSQL 10测试。

```sql
SELECT "version"()
# PostgreSQL 10.11, compiled by Visual C++ build 1800, 64-bit
```

## 继承

我们用动物(animal)和人类(human)的例子来进行说明，human继承自animal。

```sql
CREATE TABLE animal (
	"id" SERIAL,
	"name" VARCHAR(255),
	"age" INT
)
CREATE TABLE human (
	"lang" varchar(255)
) INHERITS (animal)
```

动物模型中有姓名和年龄，假设只有人类才有语言。

插入一些数据，

```sql
INSERT INTO animal ("name", "age")
VALUES ('Cat 1', 3)
INSERT INTO animal ("name", "age")
VALUES ('Cat 2', 2)
INSERT INTO human ("name", "age", "lang")
VALUES ('Leon', 34, 'Chinese')
```

查询所有动物

```sql
SELECT * FROM animal 
-- 1	Cat 1	3
-- 2	Cat 2	2
-- 3	Leon	34
```

只查询人类

```sql
SELECT * FROM ONLY human
-- 3	Leon	34	Chinese
```

只查询动物

```sql
SELECT * FROM ONLY animal 
-- 1	Cat 1	3
-- 2	Cat 2	2
```

## 注意点

1. 继承不会自动地将来自`INSERT`或`COPY`命令的数据传播到继承层次中的其他表中。

   ```sql
   SELECT p.relname, c.name, c.age
   FROM animal c, pg_class p
   WHERE c.tableoid = p.oid
   -- animal	Cat 1	3
   -- animal	Cat 2	2
   -- human	Leon	34
   ```

2. 父表上的所有检查约束和非空约束都将自动被它的后代所继承， 除非使用 `NO INHERIT`子句明确指定。其他类型的约束（唯一、主键和外键约束）则不会被继承。

3. 一个表可以从超过一个的父表继承，在这种情况下它拥有父表们所定义的列的并集。

4. 并非所有的SQL命令都能工作在继承层次上。

## Reference

- https://www.postgresql.org/docs/10/index.html

