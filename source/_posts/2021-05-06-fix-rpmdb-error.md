---
title: 解决yum install时出现Thread died in Berkeley DB library的错误
toc: false
date: 2021-05-06 16:39:00
description: ...
tags:
- CentOS
- Linux
---

今天在CentOS 7安装Wireshark时，出现了如下错误：

```shell
$ yum install wireshark wireshark-gnome
error: rpmdb: BDB0113 Thread/process 3336/140141695063872 failed: BDB1507 Thread died in Berkeley DB library
error: db5 error(-30973) from dbenv->failchk: BDB0087 DB_RUNRECOVERY: Fatal error, run database recovery
error: cannot open Packages index using db5 -  (-30973)
error: cannot open Packages database in /var/lib/rpm
CRITICAL:yum.main:

Error: rpmdb open failed
```

提示rpm相关的数据库损坏了

**解决方案**

```shell
$ mkdir /var/lib/rpm/backup
$ cp -a /var/lib/rpm/__db* /var/lib/rpm/backup/
$ rm -f /var/lib/rpm/__db.[0-9][0-9]*
$ rpm --quiet -qa
$ rpm --rebuilddb
$ yum clean all
```

## Reference

- https://cloudlinux.zendesk.com/hc/en-us/articles/115004075294-Fix-rpmdb-Thread-died-in-Berkeley-DB-library

