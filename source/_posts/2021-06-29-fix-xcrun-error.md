---
title: 解决MacOS下运行Go项目出现的xcrun error invalid active developer path问题
toc: false
date: 2021-06-29 16:30:00
description: ...
tags:
- Go
- MacOS
---

把上周在公司摸鱼玩的Go项目在家里的Mac电脑上跑一下，出现了下面的错误

```shell
$ go run .
# runtime/cgo
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

解决方案

```shell
$ xcode-select --install
$ sudo xcode-select -switch /
```

