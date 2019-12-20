---
title: QPSQL：current transaction is aborted, commands ignored until end of transaction block
toc: false
date: 2019-12-20 15:20:27
description: PostgreSQL数据库，在特定情况下，Qt调用QSqlQuery::lastInsertId()会出现 current transaction is aborted, commands ignored until end of transaction block的错误。为什么呢？
tags:
- Qt
- PostgreSQL
---

记一次愉快的问题排查经历。-_-

同事反馈给我一个问题，我们的某一个业务模块在特定操作下会出现 Unable to free statement: ERROR: current transaction is aborted, commands ignored until end of transaction block 的错误，从而导致插入数据到数据库失败。

**Step 1**

确认问题，它真的是一个问题吗？是！

**Step 2**

尝试在自己的开发环境下重现，重现了！

**Step 3**

排查业务模块的代码实现，看有没有明显的问题。没有！

结合其他信息猜测原因，个人比较喜欢通过多个信息的关联性猜测可能的原因，然后用排除法。

**Step 4**

猜测原因失败。

Google!

Google出的可能的原因都在猜测范围内，大多是说上一个事务出错了没有rollback。

但是情况并不是这样。

**Step 5**

同事反馈只要`iReturnLastInsertId`为false, 就是正常的。非常关键的信息。

我们调用的是公司封装的数据库操作接口，大概长这样（出于代码安全的考虑，省略了大部分代码）：

```c++
QVariant insertRow(const QString &iTableStr,
                   const QVariantMap &iDataMap,
                   const QStringList &iFields,
                   bool iReturnLastInsertId, 
                   const QString &iReturnField)
{
    QString sqlStr = QString("INSERT INTO %1 (%2) VALUES (%3)")
            .arg(iTableStr)
            .arg(bindFieldLst.join(','))
            .arg(bindValueLst.join(','));
    // sqlite不支持RETURNING
    if (!iReturnField.isEmpty() && (databaseType() != "QSQLITE")) {
        sqlStr += QString(" RETURNING %1").arg(iReturnField);
    }
    query.prepare(sqlStr);
    query.exec();
    QVariant result;
    if (iReturnLastInsertId) {
        result = query.lastInsertId();
    }
    commit();
    return result;
}
```

大意是封装一个支持多个数据库的接口，通过传入的参数生成裸SQL。

如果`iReturnLastInsertId`为false, 则不会调用`lastInsertId()`。

**Step 6**

写一个最小的DEMO, 只处理这一个逻辑，以此排除其他因素的影响，也更方便排除问题。

然后注释掉`lastInsertId()`调用的地方，运行正常。

**Step 7**

查看Qt官方帮助文档中关于lastInsertId的说明，如下：

[Q](../qtcore/qvariant.html)Variant QSqlQuery::lastInsertId() const

Returns the object ID of the most recent inserted row if the database supports it. An invalid [QVariant](../qtcore/qvariant.html) will be returned if the query did not insert any value or if the database does not report the id back. If more than one row was touched by the insert, the behavior is undefined.

For MySQL databases the row's auto-increment field will be returned.

Note: For this function to work in PSQL, the table table must contain OIDs, which may not have been created by default. Check the default_with_oids configuration variable to be sure.

然后再查看，PostgreSQL官方文档中关于default_with_oids 的说明，如下：

`default_with_oids` (`boolean`)

This controls whether `CREATE TABLE` and `CREATE TABLE AS` include an OID column in newly-created tables, if neither `WITH OIDS` nor `WITHOUT OIDS` is specified. It also determines whether OIDs will be included in tables created by `SELECT INTO`. The parameter is `off` by default; in PostgreSQL 8.0 and earlier, it was `on` by default.

The use of OIDs in user tables is considered deprecated, so most installations should leave this variable disabled. Applications that require OIDs for a particular table should specify `WITH OIDS` when creating the table. This variable can be enabled for compatibility with old applications that do not follow this behavior.

文档说PostgreSQL8.0之后default_with_oids都为off了，然后确认了业务模块插入的表结果。

感觉是Qt文档写错了（或者是我没理解到）。

**Step 8**

结合其他信息，这个封装的insertRow接口被广泛使用，现在才暴露问题，证明在大多数情况下都是正常的。

于是在编写的最小DEMO进行了验证。

发现：只要表结构的id的默认值使用了序列的，都正常。

而插入失败的业务表id 的默认值长这样

```
id BIGINT DEFAULT (to_char(now(), 'YYMMDDHH24MISSMS'::text) || to_char(random() * (999 - 1)::double precision + 1::double precision, 'FM009'::text))::bigint NOT NULL,
```

之所以长这样，是为了方便按日期进行分表。

**Step 9**

查看Qt源码中QPSQL的lastInsertId的实现，长这样：

```c++
QVariant QPSQLResult::lastInsertId() const
{
    Q_D(const QPSQLResult);
    if (d->privDriver()->pro >= QPSQLDriver::Version81) {
        QSqlQuery qry(driver()->createResult());
        // Most recent sequence value obtained from nextval
        if (qry.exec(QLatin1String("SELECT lastval();")) && qry.next())
            return qry.value(0);
    } else if (isActive()) {
        Oid id = PQoidValue(d->result);
        if (id != InvalidOid)
            return QVariant(id);
    }
    return QVariant();
}
```

FUCK! 当PostgreSQL 8.1 之后，lastInsertId实际执行的是`lastval()`.

PostgreSQL官方文档中关于lastval的说明：

 Return value most recently obtained with `nextval` for any sequence 。

即返回最近一次用`nextval`获取的任何序列的值 。

**Step 10**

破案！

当往一个默认ID不使用序列自增的业务表插入数据后，调用lastval()出错了！

## Reference

-  https://www.postgresql.org/docs/10/runtime-config-compatible.html#GUC-DEFAULT-WITH-OIDS 
-  https://www.postgresql.org/docs/10/functions-sequence.html 
-  https://doc.qt.io/qt-5/qsqlquery.html#lastInsertId 