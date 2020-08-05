---
title: CentOS安装SQL SERVER驱动
toc: false
date: 2019-11-24 16:41:27
description: CentOS安装SQL Server驱动, 以能访问SQL Server数据库
tags:
- CentOS
- MSSQL
---
## 方法一：安装微软官方驱动msodbcsql17

```bash
sudo su

#Download appropriate package for the OS version
#Choose only ONE of the following, corresponding to your OS version

#RedHat Enterprise Server 6
curl https://packages.microsoft.com/config/rhel/6/prod.repo > /etc/yum.repos.d/mssql-release.repo

#RedHat Enterprise Server 7
curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-release.repo

exit
sudo yum remove unixODBC-utf16 unixODBC-utf16-devel #to avoid conflicts
sudo ACCEPT_EULA=Y yum install msodbcsql17
# optional: for bcp and sqlcmd
sudo ACCEPT_EULA=Y yum install mssql-tools
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc
# optional: for unixODBC development headers
sudo yum install unixODBC-devel
```

### Qt Coding

测试代码:

```c++
#include <QCoreApplication>
#include <QtSql>
#include <QtCore>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QString databaseName = "test_db";
    QString hostName = "140.197.105.14";
    // 下面的driver也可以写成ODBC Driver 17 for SQL Server
    // ODBC Driver 17 for SQL Server是/etc/odbcinst.ini的驱动配置名称
    // 可以通过odbcinst -j查看配置文件的路径
    QString driver = "/opt/microsoft/msodbcsql17/lib64/libmsodbcsql-17.4.so.2.1";

    QSqlDatabase db = QSqlDatabase::addDatabase("QODBC");
    db.setHostName(hostName);
    db.setDatabaseName(QString("DRIVER=%1;SERVER=%2;DATABASE=%3")
                       .arg(driver)
                       .arg(hostName)
                       .arg(databaseName)
                       );
    db.setUserName("sa");
    db.setPassword("Password1");
    bool ok = db.open();
    if (!ok) {
        qWarning() << "connect to database failed." << db.lastError().text();
        return -1;
    }
    qInfo() << "connect to database done.";

    QSqlQuery query(db);
    query.prepare("SELECT 1");
    query.exec();
    if (query.next()) {
        qDebug() << "query result: " << query.value(0).toInt();
    }

    return a.exec();
}
```

测试结果：

```
connect to database done.
query result:  1
```

## 方法二：使用unixODBC+FreeTDS

unixODBC是ODBC的驱动管理器。

TDS是一个应用层协议，FreeTDS是实现TDS协议的一个免费软件。

### 安装unixODBC(如果需要的话)

```bash
# install unixODBC
wget http://www.unixodbc.org/unixODBC-2.3.7.tar.gz
tar xvzf unixODBC-2.3.7.tar.gz
cd unixODBC-2.3.7/
./configure --prefix=/usr/local/unixODBC
make
make install
echo 'export PATH="$PATH:/usr/local/unixODBC/bin"' >> ~/.bash_profile
echo 'export PATH="$PATH:/usr/local/unixODBC/bin"' >> ~/.bashrc
source ~/.bashrc
```

### 安装FreeTDS

```bash
# install FreeTDS
wget ftp://ftp.freetds.org/pub/freetds/stable/freetds-1.1.1.tar.gz
tar xvzf freetds-1.1.1.tar.gz 
cd freetds-1.1.1/
./configure --prefix=/usr/local/freetds --with-tdsver=7.4 --enable-msdblib --with-unixodbc=/usr/local/unixODBC
make
make install
echo 'export PATH="$PATH:/usr/local/freetds/bin"' >> ~/.bash_profile
echo 'export PATH="$PATH:/usr/local/freetds/bin"' >> ~/.bashrc
source ~/.bashrc
```

打印unixODBC配置信息

```bash
[root@localhost etc]# odbcinst -j
unixODBC 2.3.7
DRIVERS............: /etc/odbcinst.ini
SYSTEM DATA SOURCES: /etc/odbc.ini
FILE DATA SOURCES..: /etc/ODBCDataSources
USER DATA SOURCES..: /root/.odbc.ini
SQLULEN Size.......: 8
SQLLEN Size........: 8
SQLSETPOSIROW Size.: 8
```

打印FreeTDS配置信息

```bash
[root@localhost etc]# tsql -C
Compile-time settings (established with the "configure" script)
                            Version: freetds v1.1.1
             freetds.conf directory: /usr/local/freetds/etc
     MS db-lib source compatibility: yes
        Sybase binary compatibility: no
                      Thread safety: yes
                      iconv library: yes
                        TDS version: 7.4
                              iODBC: no
                           unixodbc: yes
              SSPI "trusted" logins: no
                           Kerberos: no
                            OpenSSL: no
                             GnuTLS: no
                               MARS: yes
```

### 配置驱动 `odbcinst.ini`

```ini
[FreeTDS]
Description=FreeTDS driver
Driver=/usr/local/freetds/lib/libtdsodbc.so.0
```

###`isql`测试

`isql`是unixODBC的一个测试命令行工具。

配置ODBC数据源 `odbc.ini`

```ini
[TEST_DS_1]
Driver  = FreeTDS
Description = Nothing
Trace = No
Server = 139.196.104.13
Port = 1433
Database = s_cell
```

测试

```bash
[root@localhost ~]# isql -v TEST_DS_1 sa 123456
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> select 1
+------------+
|            |
+------------+
| 1          |
+------------+
SQLRowCount returns 1
1 rows fetched
```

### 测试`tsql`

`tsql`是FreeTDS的一个命令行测试工具。

配置FreeTDS `freetds.conf`

```ini
[Test_DS_2]
        host = 139.196.104.13
        port = 1433
        tds version = 7.4
```

测试

```bash
[root@localhost ~]# tsql -S TEST_DS_2 -U sa -P 123456
locale is "en_US.UTF-8"
locale charset is "UTF-8"
using default charset "UTF-8"
1> select 1
2> go

1
(1 row affected)
```

### Qt Coding

测试代码 :

```c++
#include <QCoreApplication>
#include <QtSql>
#include <QtCore>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QString databaseName = "test_db";
    QString hostName = "140.197.105.14";
    QString driver = "/usr/local/freetds/lib/libtdsodbc.so.0"; // Or FreeTDS

    QSqlDatabase db = QSqlDatabase::addDatabase("QODBC");
    db.setHostName(hostName);
    db.setDatabaseName(QString("DRIVER=%1;SERVER=%2;DATABASE=%3")
                       .arg(driver)
                       .arg(hostName)
                       .arg(databaseName)
                       );
    db.setPort(1433);
    db.setUserName("sa");
    db.setPassword("Password1");
    bool ok = db.open();
    if (!ok) {
        qWarning() << "connect to database failed." << db.lastError().text();
        return -1;
    }
    qInfo() << "connect to database done.";

    QSqlQuery query(db);
    query.prepare("SELECT 1");
    query.exec();
    if (query.next()) {
        qDebug() << "query result: " << query.value(0).toInt();
    }

    return a.exec();
}
```

## 可能出现的问题

不同的驱动支持的数据库版本可能有差异。

## Reference

- https://docs.microsoft.com/zh-cn/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-2017
- http://www.unixodbc.org/doc/FreeTDS.html
- https://stackoverflow.com/questions/31980980/difference-between-freetds-and-unixodbc