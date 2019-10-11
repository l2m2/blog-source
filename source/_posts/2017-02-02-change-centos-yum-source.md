---
title: CentOS 7 切换yum源为阿里云源
toc: false
date: 2017-02-02 16:18:27
description: 祖国万岁
categories:
- 锤子的锤
tags:
- CentOS
---
```bash
# 备份
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 下载
sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 运行yum makecache生成缓存
sudo yum clean all
sudo yum makecache
```
