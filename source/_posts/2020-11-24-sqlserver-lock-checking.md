---
title: SQL Server中的阻塞与死锁
toc: true
date: 2020-11-24 10:59:00
description: 记录排查过程
tags:
- MSSQL
---

## 遇到的问题

客户反馈在打开某模块时，界面一直停留在加载中。

查看该模块对应的数据库表的锁情况。

```sql
SELECT * FROM sys.dm_tran_locks
WHERE resource_database_id = DB_ID() 
  AND resource_associated_entity_id = OBJECT_ID(N'dbo.utclabm_daily_job')
```

此时，我们可以KILL掉会话以恢复。

```sql
KILL 85 -- 85 为request_session_id列的值
```

为了找到根本原因，我们不着急KILL掉会话。

继续查看是什么引起了阻塞。

```sql
SELECT
	t1.resource_type [资源锁定类型],
	DB_NAME( resource_database_id ) 数据库名,
	t1.resource_associated_entity_id [锁定对象],
	t1.request_mode [等待者请求的锁定模式],
	t1.request_session_id [等待者SID],
	t2.wait_duration_ms [等待时间],
	( SELECT TEXT FROM sys.dm_exec_requests r CROSS apply sys.dm_exec_sql_text ( r.sql_handle ) WHERE r.session_id = t1.request_session_id ) [等待者要执行的SQL],
	t2.blocking_session_id [锁定者SID],
	( SELECT TEXT FROM sys.sysprocesses p CROSS apply sys.dm_exec_sql_text ( p.sql_handle ) WHERE p.spid = t2.blocking_session_id ) [锁定者执行的SQL] 
FROM
	sys.dm_tran_locks t1,
	sys.dm_os_waiting_tasks t2 
WHERE
	t1.lock_owner_address = t2.resource_address
```

查询结果：

![](/images/sqlserver-lock-checking-1.png)

可以看到，会话69和会话73形成了死锁。

会话69执行的SQL是

```sql
SELECT * FROM (
    SELECT ROW_NUMBER() OVER (ORDER BY "titration_volume" ASC ,"plan_time" ASC ) AS row__number, "id","group_id","workcenter_id","line_id","bath_id","control_item","control_grade","spec_unit","frequency","spec_lower_limit","spec_control_point","spec_upper_limit","solution_name","lab_concentration","analysis_type","analysis_value","stopping","titration_volume","4_distribution","analysis_operator","sample_time","analysis_time","formula_method","time_value","time_unit","obsolete","weight","plan_time","stop_reason","review_volume","review_weight","review_result","review_operator","review_time","handle_time","wx_send_info","create_time" 
    FROM "utclabm_daily_job"  
    WHERE  (  ( "group_id" LIKE '%化金%' OR "workcenter_id" LIKE '%化金%' OR "line_id" LIKE '%化金%' OR "bath_id" LIKE '%化金%' OR "control_item" LIKE '%化金%' OR "spec_unit" LIKE '%化金%' OR "solution_name" LIKE '%化金%' OR "review_operator" LIKE '%化金%' )  AND  (  (  (  (  ( "plan_time" >= '2020-12-17 08:00:00' )  AND  ( "plan_time" <= '2020-12-17 19:59:59' )  )  )  OR  (  (  (  (  ( "4_distribution" LIKE '%D%' OR "4_distribution" LIKE '%C%' )  AND "review_result" IS  NULL  )  )  OR  ( "analysis_value" IS  NULL  )  )  )  )  )  ) 
) AS ROWNUM__ALIAS__TABLE WHERE  ROWNUM__ALIAS__TABLE.row__number < 101
```

会话73执行的SQL是

```sql
(@P1 nvarchar(18),@P2 nvarchar(4),@P3 nvarchar(2),@P4 nvarchar(8),@P5 nvarchar(4),@P6 nvarchar(4),@P7 nvarchar(10),@P8 nvarchar(4000),@P9 nvarchar(4000),@P10 nvarchar(4000),@P11 nvarchar(4000),@P12 nvarchar(4),@P13 nvarchar(4000),@P14 nvarchar(4000),@P15 nvarchar(4000),@P16 nvarchar(4000),@P17 nvarchar(4000),@P18 ntext)
UPDATE "utclabm_daily_job" 
SET "analysis_type"=@P1,"analysis_value"=@P2,"stopping"=@P3,"stop_reason"=@P4,"titration_volume"=@P5,"4_distribution"=@P6,"analysis_operator"=@P7,"sample_time"=@P8,"analysis_time"=@P9,"formula_method"=@P10,"obsolete"=@P11,"weight"=@P12,"review_volume"=@P13,"review_weight"=@P14,"review_result"=@P15,"review_operator"=@P16,"review_time"=@P17,"log"=@P18 
WHERE  (  ( "id" = '496126' )  ) 
```

也可以直接查询[sysprocesses](https://docs.microsoft.com/en-us/sql/relational-databases/system-compatibility-views/sys-sysprocesses-transact-sql?view=sql-server-ver15)表查看这两个会话的详细信息。

```sql
select cmd, spid, blocked, login_time, last_batch, hostname, lastwaittype
from sys.sysprocesses
where blocked > 0 and spid in (69, 73)
--SELECT          	69	73	2020-12-17 00:23:50.367	2020-12-17 02:16:27.247	PC-A-D430-N084                                                                                                                  	LCK_M_S                         
--UPDATE          	73	69	2020-12-16 11:57:39.683	2020-12-17 02:16:27.790	SERVER02-PC                                                                                                                     	LCK_M_IX                        
```

下图是各种锁模式的[兼容性](https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms186396(v=sql.105)?redirectedfrom=MSDN)。

![](/images/sqlserver-lock-checking-2.png)



那什么是阻塞？什么又是死锁呢？

## 阻塞

我们举个例子来理解阻塞。

我们假定我们有两个数据库用户UserA和UserB。

UserA 正在编辑发票表Invoice, 执行的SQL可能长这样：

 ```sql
UPDATE Innoive 
SET xx = ...
WHERE InvoiveId = ...
 ```

执行此操作，SQL Server数据库内部的会话关联的线程必须获取并保持以下两个锁：

- Invoice表和页上的IX Lock (意向排他锁) ，包含UserA正在编辑的行。此锁用于建议锁层次结构以执行数据修改。
- UserA正在编辑行的X Lock (排他锁)。这意味着在释放该锁之前，当前会话是唯一允许修改此行的会话。

在同一时刻，UserB 想获取当前月份的发票列表。不巧的是，UserA编辑的行刚好在这个列表内。UserB所在会话关联的线程将要：

- 获取Inovice表上的IS Lock (意向共享锁)。此锁用于建立锁层次结构，以执行只读操作。
- 在Inovice列表所需的页上尝试 S Lock (共享锁)。其中，UserA获取了该页的X锁，X和S是不兼容的，所以负责UserB会话的线程必须等待UserA的会话释放X锁。在释放之前，UserB的会话被UserA的会话**阻塞**了。

如下图：

![](/images/sqlserver-lock-checking-3.png)

当`UPDATE`执行完成, UserA提交事务后，这个阻塞就不存在了。

在实际业务中，我们遇到的情况要比这个复杂得多，远不止两个会话。如下图：

![](/images/sqlserver-lock-checking-4.png)

这上图的例子中，我们可以看到(59, 79, 145)这三个阻塞的会话。但145实际是被59阻塞的，所以一共有两个顶层阻塞会话(59, 79), 其他的会话我们称之为等待者。

## 死锁

死锁和阻塞的原理相同，不同的是发生死锁时，没有可识别的顶部阻塞会话，会形成一个封闭的锁环。

为了更好的理解，我们继续用刚才的发票的例子来说明。

假设在修改Invoice表的行数据时，UserA还需要从InvoiceDetails表中获取向客户开票的总金额。假设由于我们尚未得知的原因，UserB已经在包含UserA需要读取的InvoiceDetails表行的页上获取了S Lock。如下图：

![](/images/sqlserver-lock-checking-5.png)

如上图所示，两个会话都在等待永远不会释放的锁。



## Reference

- https://stackoverflow.com/questions/694581/how-to-check-which-locks-are-held-on-a-table
- https://www.sqlshack.com/locking-sql-server/
- https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql?view=sql-server-ver15
- https://www.cnblogs.com/88223100/p/SQL_SERVER_Optimization_Statement.html
- https://zhuanlan.zhihu.com/p/54000324
- https://www.sqlshack.com/what-are-sql-server-deadlocks-and-how-to-monitor-them/

