---
title: SVN迁移到Git
toc: false
date: 2020-03-25 14:33:00
description: 保留所有提交记录
tags:
- Git
---

1. 创建SVN用户与Git用户的映射关系文件user.txt. 文件内容如下：

   ```
   svnuser1=gituser1<gituser1@xxx.com>
   svnuser2=gituser2<gituser2@xxx.com>
   ```

   在Windows Power Shell下执行下面的命令，可以获取到svn库的所有提交的用户：

   ```powershell
   svn.exe log --quiet |
   ? { $_ -notlike '-*' } |
   % { ($_ -split ' \| ')[1] } |
   Sort -Unique
   ```

2. 用`git svn`从SVN库拉取数据

   ```bash
   $ git svn clone --no-metadata -A users.txt http://svn.xxx.net/svn/svnproject gitproject \
   --user leon --log-window-size 5000
   ```

   users.txt是步骤1的用户映射关系文件路径。

   http://svn.xxx.net/svn/svnproject 是svn的地址。

   gitproject是本地git仓库的名称。

   --user leon是指定svn的用户 。如果本地没有登陆过该用户，会弹出提示框输入密码。

3. 拉取完成后可以通过`git log`看到日志

4. Push到远程仓库

   ```bash
   $ git remote add origin git@gitlab.xxx.net:jsc/jscc04.git
   $ git push -u origin master
   ```

## Reference

- https://www.lovelucy.info/codebase-from-svn-to-git-migration-keep-commit-history.html
- https://stackoverflow.com/questions/2494984/how-to-get-a-list-of-all-subversion-commit-author-usernames

- https://git-scm.com/docs/git-svn

