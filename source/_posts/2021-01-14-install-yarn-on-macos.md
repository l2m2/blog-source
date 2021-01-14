---
title: MacOS安装yarn
toc: false
date: 2021-01-14 15:48:00
description: ...
tags:
- MacOS
- yarn
---

## Yarn

yarn是与npm类似的一个JavaScript包管理工具。

## 安装

```bash
$ brew install yarn
```

若提示错误 Error: An exception occurred within a child process: CompilerSelectionError: yarn cannot be built with any available compilers.需要先装gcc

```bash
$ brew install gcc
```

安装gcc过程中若提示错误Error: gcc: the bottle needs the Xcode CLT to be installed.可执行下面的命令

```bash
$ xcode-select --install
```

## 使用

```bash
$ yarn global add @vue/cli
$ yarn global upgrade --latest @vue/cli
$ yarn run build
$ yarn run serve
```

## Reference

- https://www.sitepoint.com/yarn-vs-npm/
- https://zhuanlan.zhihu.com/p/23493436

