---
title: CentOS 6 安装 Python3.6
toc: true
date: 2020-01-09 11:09:38
description: Fuck CentOS 6.
tags:
- CentOS
- Python
---

CentOS 6上默认的Python版本是2.x，需要安装Python 3.6+。

## 安装Python3.6

有很多安装方法，此处只列举其中两种: 

### using Software Collections Repository (SCL)。

```bash
[root@ci-centos6 home]# yum install centos-release-scl
[root@ci-centos6 home]# yum install rh-python36
```

检查是否安装成功：

```bash
[root@ci-centos6 home]# yum info rh-python36
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * centos-sclo-rh: mirrors.163.com
 * centos-sclo-sclo: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Installed Packages
Name        : rh-python36
Arch        : x86_64
Version     : 2.0
Release     : 1.el6
Size        : 0.0  
Repo        : installed
From repo   : centos-sclo-rh
Summary     : Package that installs rh-python36
License     : GPLv2+
Description : This is the main package for rh-python36 Software Collection.
```

使用：

```bash
[root@ci-centos6 home]# scl enable rh-python36 bash
[root@ci-centos6 home]# python --version
Python 3.6.9
```

**注意：** 这种方法你无法在bash之外使用python3。如果你切换了Terminal, 你的Python版本会恢复为之前的2.x.

**修改默认的Python为Python3.6**

```bash
[root@ci-centos6 ~]# echo "alias python='/opt/rh/rh-python36/root/usr/bin/python'" >> /root/.bashrc
[root@ci-centos6 ~]# echo "alias pip='/opt/rh/rh-python36/root/usr/bin/pip'" >> /root/.bashrc
[root@ci-centos6 ~]# source /root/.bashrc
[root@ci-centos6 ~]# python --version
Python 3.6.9
```

### 编译安装

下载源码

```bash
[root@ci-centos6 ~]# wget https://www.python.org/ftp/python/3.6.9/Python-3.6.9.tgz
```

解压

```bash
[root@ci-centos6 ~]# tar xvzf Python-3.6.9.tgz 
```

编译

```bash
[root@ci-centos6 Python-3.6.9]# ./configure --prefix=/usr/local/python3.6 --enable-optimizations
[root@ci-centos6 Python-3.6.9]# make -j4
[root@ci-centos6 Python-3.6.9]# make install
```

## Refercence

-  https://www.2daygeek.com/install-python-3-on-centos-6/ 