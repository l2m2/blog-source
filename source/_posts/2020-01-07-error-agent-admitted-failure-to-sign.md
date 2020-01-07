---
title: Error:Agent admitted failure to sign
toc: false
date: 2020-01-07 10:45:24
description: Git Clone时出现Error:Agent admitted failure to sign的解决办法
tags:
- Git
---

在某些情况下，在Linux上通过ssh连接github/gitlab会产生错误“Agent admitted failure to sign using the key”.

```bash
[root@192 gitlab]# git clone git@gitlab.xxx.net:leon.li/python-demo
Cloning into 'python-demo'...
Agent admitted failure to sign using the key.
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

**解决方法**

```bash
[root@192 gitlab]# eval "$(ssh-agent -s)"
Agent pid 15383
[root@192 gitlab]# ssh-add
Identity added: /root/.ssh/id_rsa (/root/.ssh/id_rsa)
[root@192 gitlab]# git clone git@gitlab.xxx.net:leon.li/python-demo
Cloning into 'python-demo'...
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 9 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (9/9), done.
```

## Reference

- [Github Help: Error: Agent admitted failure to sign](https://help.github.com/en/github/authenticating-to-github/error-agent-admitted-failure-to-sign)

