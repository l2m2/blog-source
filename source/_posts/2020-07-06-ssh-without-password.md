---
title: SSH免密登录
toc: false
date: 2020-07-01 15:04:00
description: ssh-keygen & ssh-copy-id
tags:
- Linux
---

1. 通过`ssh-keygen`生成公钥和私钥

   如果已有。则可以跳过此步骤。

   ```bash
   $ ssh-keygen -t rsa -b 4096 -C "l2m2 home mac"
   ```

   生成后，在`~/.ssh/`目录下将有以下文件

   ```bash
   $ ls
   id_rsa  id_rsa.pub
   ```

2. 通过`ssh-copy-id`拷贝公钥到远程主机

   ```bash
   $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@140.197.105.14
   ```

   过程中按照提示输入远程主机的密码

3. 现在可以免密登录远程主机了。

   ```bash
   $ ssh root@140.197.105.14
   Last login: Mon Jul  6 12:59:37 2020 from 118.112.72.21
   
   Welcome to Alibaba Cloud Elastic Compute Service !
   
   -bash: export: `=': not a valid identifier
   -bash: export: `/opt/node-v6.10.3-linux-x64': not a valid identifier
   -bash: export: `=': not a valid identifier
   -bash: export: `/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin': not a valid identifier
   [root@iZuf61ti9rpeooj054hbpgZ ~]#
   ```

## Reference

- https://www.thegeekstuff.com/2008/11/3-steps-to-perform-ssh-login-without-password-using-ssh-keygen-ssh-copy-id/

