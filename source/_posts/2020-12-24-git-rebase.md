---
title: Git Rebase
toc: false
date: 2020-12-24 13:53:00
description: 通过一些场景来说明rebase的作用。
tags:
- Git
---

通过一个例子来说明rebase的作用。

从字面上看来讲，rebase就是re+base，翻译过来大概是重新定义分支的参考基准。

base就是指这个分支是从哪里生出来的。

## 场景一：在过时的分支上开发, 日志分叉了

我们新建了一个工程dirty，只有一个README.md。

分别clone到本地不同的两个目录，以模拟多人协作。

A

```bash
$ ls
README.md
$ pwd
/e/dirty
```
B
```bash
$ ls
README.md
$ pwd
/f/workspace/toplinker/gitlab/leon.li/dirty
```

A新建一个a.txt, push到远程。

```bash
$ echo a >> a.txt
$ git add .
$ git commit -m "a add txt"
$ git push
```

然后B再新建一个b.txt, push到远程。

```bash
$ echo b >> b.txt
$ git add .
$ git commit -m "b add txt"
$ git push
To gitlab.topibd.net:leon.li/dirty.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'gitlab.topibd.net:leon.li/dirty.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

B push到远程时发生错误，提醒我们要先pull。

```bash
$ git pull
```

没啥问题，B来看看日志。

```bash
$ git log --graph --pretty=oneline --abbrev-commit
*   4e12fc2 (HEAD -> master) Merge branch 'master' of gitlab.topibd.net:leon.li/dirty
|\
| * bc77cc9 (origin/master, origin/HEAD) a add txt
* | 26b7898 b add txt
|/
* 2565f23 Initial commit
```

**提交历史分叉了。不太舒服，这个时候我们可以使用rebase。**

```bash
$ git rebase
Successfully rebased and updated refs/heads/master.
$ git log --graph --pretty=oneline --abbrev-commit
* d47fde7 (HEAD -> master) b add txt
* bc77cc9 (origin/master, origin/HEAD) a add txt
* 2565f23 Initial commit
```

提交历史的分叉没有了。

## Reference

- https://git-scm.com/docs/git-rebase
- https://www.liaoxuefeng.com/wiki/896043488029600/1216289527823648
- https://gitbook.tw/chapters/branch/merge-with-rebase.html

