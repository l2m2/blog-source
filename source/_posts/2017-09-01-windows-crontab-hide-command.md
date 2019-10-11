---
title: Windows任务计划执行BAT时隐藏窗口
toc: false
date: 2017-09-01 16:04:41
description: 消除小黑窗
tags:
- Windows
---
- 添加VBS脚本，示例如下

  ```vbscript
  DIM objShell
  set objShell=wscript.createObject("wscript.shell")
  iReturn=objShell.Run("cmd.exe /C C:/fuck/bin/fuck.exe fuck.param", 0, TRUE)
  ```

- 任务计划中执行此VBS脚本