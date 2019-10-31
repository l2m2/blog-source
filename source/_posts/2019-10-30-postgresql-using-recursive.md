---
title: PostgreSQL查询中使用递归
toc: false
date: 2019-10-30 17:18:31
description: 你可能不知道的SQL
tags:
- PostgreSQL
- SQL
---

## WITH查询

官网对`WITH`的解释：

WITH provides a way to write auxiliary statements for use in a larger query. These statements, which are often referred to as Common Table Expressions or CTEs, can be thought of as defining temporary tables that exist just for one query. Each auxiliary statement in a WITH clause can be a SELECT, INSERT, UPDATE, or DELETE; and the WITH clause itself is attached to a primary statement that can also be a SELECT, INSERT, UPDATE, or DELETE.

 `WITH`提供了一种方式来书写在一个大型查询中使用的辅助语句。这些语句通常被称为公共表表达式或CTE，它们可以被看成是定义只在一个查询中存在的临时表。在`WITH`子句中的每一个辅助语句可以是一个`SELECT`、`INSERT`、`UPDATE`或`DELETE`，并且`WITH`子句本身也可以被附加到一个主语句，主语句也可以是`SELECT`、`INSERT`、`UPDATE`或`DELETE`。 

我们来看一个例子。

```plsql
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region
), top_regions AS (
    SELECT region
    FROM regional_sales
    WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)
SELECT region,
       product,
       SUM(quantity) AS product_units,
       SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```

 它只显示在高销售区域每种产品的销售总额。`WITH`子句定义了两个辅助语句`regional_sales`和`top_regions`，其中`regional_sales`的输出用在`top_regions`中而`top_regions`的输出用在主`SELECT`查询。这个例子可以不用`WITH`来书写，但是我们必须要用两层嵌套的子`SELECT`。使用这种方法要更简单些。 

## 递归查询

下面的例子是计算从1到100的整数的总和的查询。

```plsql
WITH RECURSIVE t(n) AS (
    VALUES (1)
  UNION ALL
    SELECT n+1 FROM t WHERE n < 100
)
SELECT sum(n) FROM t;
```

我们再来玩一个产生[斐波那契数列](https://en.wikipedia.org/wiki/Fibonacci_number)的例子。

斐波那契数列由0和1开始，之后的斐波那契系数就是由之前的两数相加而得出。

注意"0"是省略掉的，所以数列长这个样子：

(0, ), 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, ...

```plsql
WITH RECURSIVE r ( a, b ) AS ( 
	SELECT 0::INT, 1::INT 
	UNION ALL 
	SELECT b, a + b FROM r 
	WHERE b < 1000 
) 
SELECT b FROM r;
```

我们可以这样理解，r (a, b) 是一个包含a, b 两列的临时表，后面的语句是为了迭代产生临时表的数据。

**我们再来看一个实际项目中的例子。**

有一张`mes_workcenter`表，用来表示MES系统中的工作中心，此处的工作中心是一个宽泛的概念，它可以是工厂，也可以是产线，机器，工位。

工作中心表有三个字段（为了更方便演示省略了其他字段），id, name, parent_id。分别表示工作中心的编号，名称，父工作中心的ID。它的数据示例如下：

| id   | name             | parent_id |
| ---- | ---------------- | --------- |
| 1    | L2M2集团         |           |
| 2    | L2M2成都有限公司 | 1         |
| 3    | 成都一厂         | 2         |
| 4    | 包装部           | 3         |
| 5    | 产线一           | 4         |
| 6    | 产线二           | 4         |

我们要查询L2M2集团下所有的工作中心，以及工作中心的子工作中心，以此类推。

```plsql
WITH RECURSIVE t(id) AS (
	SELECT id FROM mes_workcenter
	WHERE name = 'L2M2集团'
	UNION ALL
	SELECT B.id FROM t
	INNER JOIN mes_workcenter AS B ON B.parent_id = t.id
)
SELECT * FROM t
```

## 参考

-  http://www.postgres.cn/docs/10/queries-with.html 
-  https://www.postgresql.org/docs/10/queries-with.html 