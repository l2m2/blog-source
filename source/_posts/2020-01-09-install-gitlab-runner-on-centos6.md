---
title: CentOS 6安装GitLab Runner
toc: false
date: 2020-01-09 09:24:30
description: Runner Everywhere.
tags:
- GitLab
- CentOS
---

## 安装

查询当前环境的arch

```bash
[root@ci-centos6 data]# arch
x86_64
```

**下载对应的二进制包**

```bash
[root@ci-centos6 data]# curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
```

也可以在浏览器中下载好再拷贝到/usr/local/bin下

**添加权限**

```bash
[root@ci-centos6 ~]# chmod +x /usr/local/bin/gitlab-runner 
```

**创建GitLab CI用户**

```bash
[root@ci-centos6 ~]# useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
```
**安装GitLab Runner以及以服务运行**

```bash
[root@ci-centos6 home]# gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
[root@ci-centos6 home]# gitlab-runner start
```

## 注册

```bash
[root@ci-centos6 home]# gitlab-runner register --non-interactive --url "http://gitlab.xxx.net/" --registration-token "nyy7H-xxx" --executor "shell" --description "ci01-centos6_x64_192.168.2.21" --tag-list "linux,linux_x64,centos,centos6,centos6_x64" --run-untagged="true" --locked="false"
```

## Reference

-  https://docs.gitlab.com/runner/install/linux-manually.html