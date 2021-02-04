---
title: Git入门
toc: true
date: 2017-01-23 12:59:00
description: 给傻逼看的Git基础指南
tags:
- Git
---

## 1 安装Git

###  1.1 Windows

从官网[下载](https://git-scm.com/download/win), 直接默认安装即可。

##  2  从远程仓库克隆开始一个项目

###  2.1  一个基本操作的示例

GitLab上已创建了空白的仓库，先克隆远程仓库到本地

```bash
$ git clone git@gitlab.xxx.net:leon.li/git-demo.git
```

创建一个README.md文件，并提交到本地仓库。

```bash
$ cd git-demo/
$ touch README.md
$ git add README.md
$ git commit -m "create readme."
```

推送到远程master分支。

```bash
$ git push
Counting objects: 3, done.
Writing objects: 100% (3/3), 207 bytes | 103.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To gitlab.xxx.net:leon.li/git-demo.git
 * [new branch]      master -> master
```

###  2.2  创建develop分支并推送到远程

在我们现行的项目Git规范下，一般有两个分支，master和develop。

我们先看看本地和远程有哪些分支。

```bash
$ git branch -a
* master
  remotes/origin/master
```

我们先在本地创建一个develop分支。

```bash
$ git branch develop
$ git branch -a
  develop
* master
  remotes/origin/master
```

可以看到，我们已成功创建了develop分支。

先切换到develop分支。

```bash
$ git checkout develop
Switched to branch 'develop'
```

再推送到远程develop(此时远程还没有develop分支)。

```bash
$ git push -u origin develop
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: To create a merge request for develop, visit:
remote:   http://gitlab.xxx.net/leon.li/git-demo/merge_requests/new?merge_request%5Bsource_branch%5D=develop
remote:
To gitlab.xxx.net:leon.li/git-demo.git
 * [new branch]      develop -> develop
Branch 'develop' set up to track remote branch 'develop' from 'origin'.
```

### 2.3  用Git Flow创建develop分支

如果你安装了Git Flow, 那么可以以下面的姿势来玩。

```bash
$ git flow init -d 
```

此时本地已创建了develop分支，并自动切换到develop了。

再推送到远程。

```bash
$ git push -u origin develop
```

##  3 从本地仓库开始一个项目

如果项目的远程仓库还没有创建，我们可能需要从本地仓库开始一个项目。

```bash
$ mkdir git-demo2
$ cd git-demo2
$ git init
```

本地仓库建好了。

创建一个README.md，并提交到本地仓库。

```bash
$ touch README.md
$ git add README.md
$ git commit -m "create readme."
[master (root-commit) 1f72271] create readme.
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md
```

假设此时远程仓库已经建好了，添加远程仓库。

```bash
$ git remote add origin git@gitlab.xxx.net:leon.li/git-demo2.git
```

再推送到远程仓库。

```bash
$ git push -u origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 206 bytes | 206.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To gitlab.xxx.net:leon.li/git-demo2.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

## 4  常用Git命令

### 4.1 配置

```bash
# 查看git所有设置
git config --list
# 设置socks5代理
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
# 只代理github.com
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
git config --global https.https://github.com.proxy socks5://127.0.0.1:1080
# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
# 通过修改postBuffer提升git clone速度
git config --global http.postBuffer 524288000
```

### 4.2 分支

```bash
# 查看本地分支
git branch
# 查看所有分支（包括远程分支）
git branch -a
# 创建本地分支
git branch br1
# 切换分支
git checkout br1
# 创建分支并切换
git checkout -b br1
# 删除本地分支
git branch -d br1
# 删除远程分支
git push origin :br1
# 比较两个分支的差异
git diff br1..br2
# 比较当前分支与br2分支
git diff ..br2
```

### 4.3 远程仓库

```bash
# 克隆到本地
git clone git@gitlab.xxx.net:leon.li/git-demo.git
# 只克隆最近一个commit
git clone git@gitlab.xxx.net:leon.li/git-demo.git --depth=1
# 只克隆最近一个commit后拉取所有
git pull --unshallow
# 添加远程仓库
git remote add origin git@gitlab.xxx.net:leon.li/git-demo2.git
# 修改远程仓库
git remote set-url origin new.git.url/here
# 查看远程仓库
git remote show origin
# 同步远程已经删除的分支
git remote prune origin
# 提交到远程源origin的develop分支
git push origin develop
# 关联远程分支并提交，这样每次不用指定远程的源和分支，只需git pull 或 git push
git push -u origin develop
# 强制提交
git push -f
```

### 4.4 查看日志

```bash
# 查看某个文件的提交记录
git log calculate.js
git log --pretty=oneline calculate.js
# 查看最近一次提交的文件修改记录
git log --name-status HEAD^..HEAD
# 查看最近一次提交的详细记录（包括文件内容）
git log -1
git show ${commit_id}
```

### 4.5 提交/合并/撤销

```bash
# 撤销本地的所有修改
git checkout .
# 撤销本地某个文件的修改
git checkout -- book.cpp
# 撤销本地的最近一次提交
# --soft 不删除改动代码，只撤销commit
# --hard 删除改动代码，恢复到上一次的commit状态
git reset --soft HEAD^
# reset到某次commit
git reset --hard 80921b6cddb3f6465f50cadbef5bc33aeb0d48f
# 修改最近一次提交的注释
git commit --amend -m "New commit message"
# 合并，不带br1的commit历史，把不必要的commit进行压缩
git merge --squash br1
git commit -m "new commit log"
# 合并，关闭fast-forward模式，在提交的时候，会创建一个merge的commit信息
git merge --no-ff br1
```

### 4.6 submodule

```bash
# 查看submodule
git submodule
# 添加submodule到party3/libdxfrw下
git submodule add git@github.com:codelibs/libdxfrw.git party3/libdxfrw
# 更新submodule
git submodule update
# 删除submodule
git submodule deinit -f party3/libiconv
git rm -f party3/libiconv
git commit -m "removed submodule"
rm -rf .git/modules/party3/libiconv/
```

## 5. 应用场景

### 5.1 回滚到某次commit, 且不保留那次commit之后的修改

```bash
git reset --hard 80921b6cddb3f6465f50cadbef5bc33aeb0d48f
git push -f
```

