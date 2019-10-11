---
title: 解决pip install太慢的问题
toc: false
date: 2018-11-11 15:51:07
description: 解决从国外的源下载太慢的问题
categories:
- 锤子的锤
tags:
- python
---
解决从国外的源下载太慢的问题

- Windows

  ```bash
  cd ~
  mkdir pip
  cd pip
  vim pip.ini
  ```

  pip.ini

  ```ini
  [global]
  index-url=http://mirrors.aliyun.com/pypi/simple/
  [install]
  trusted-host=mirrors.aliyun.com
  ```

  

- Linux

  ```bash
  vim ~/.pip/pip.conf
  ```

  pip.conf

  ```ini
  [global]
  index-url=http://mirrors.aliyun.com/pypi/simple/
  [install]
  trusted-host=mirrors.aliyun.com
  ```

  

