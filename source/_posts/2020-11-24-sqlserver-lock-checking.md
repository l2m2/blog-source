---
title: 排查SQL Server中的锁阻塞
toc: false
date: 2020-11-24 10:59:00
description: 记录排查过程
tags:
- MSSQL
---

客户反馈在打开某模块时，界面一直停留在加载中。

查看该模块对应的数据库表的锁情况。

```sql
SELECT * FROM sys.dm_tran_locks
WHERE resource_database_id = DB_ID() 
  AND resource_associated_entity_id = OBJECT_ID(N'dbo.utclabm_daily_job')
```

如果有记录，证明有锁阻塞现象。此时，我们可以KILL掉会话以恢复。

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
	t1.request_session_id [等待者 SID],
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

可以看到，Session 77 请求的是IX锁定，即意向排他锁。其余会话请求的是S锁定，即共享锁。

下图是各种锁模式的兼容性。

![](/images/sqlserver-lock-checking-2.png)



## Reference

- https://stackoverflow.com/questions/694581/how-to-check-which-locks-are-held-on-a-table
- https://www.sqlshack.com/locking-sql-server/
- https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql?view=sql-server-ver15
- https://www.cnblogs.com/88223100/p/SQL_SERVER_Optimization_Statement.html
- https://zhuanlan.zhihu.com/p/54000324