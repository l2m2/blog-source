---
title: NSIS常见问题
toc: false
date: 2021-07-06 05:24:00
description: ...
tags:
- NSIS
---

- 给快捷方式添加启动参数

  ```nsis
   CreateShortCut "$DESKTOP\Xxx.lnk" "$INSTDIR\Application\chrome.exe" "https://l2m2.top"
  ```

  [CreateShortCut](https://nsis.sourceforge.io/Reference/CreateShortCut)的具体用法：

  ```nsis
  CreateShortCut link.lnk target.file [parameters [icon.file [icon_index_number [start_options [keyboard_shortcut [description]]]]]]
  ```

- 取消完成页的启动程序

  注释掉 `!define MUI_FINISHPAGE_RUN`所在行

  ```nsis
  ;!define MUI_FINISHPAGE_RUN "$INSTDIR\Application\chrome.exe"
  ```

- 取消License页面

  注释掉`!insertmacro MUI_PAGE_LICENSE`所在行

  ```nsis
  ;!insertmacro MUI_PAGE_LICENSE "..\..\..\path\to\licence\YourSoftwareLicence.txt"
  ```

- 执行某个批处理文件直到它退出

  ```nsis
  ExecWait '"$INSTDIR\somebat.bat"'
  ```

  [ExecWait](https://nsis.sourceforge.io/Reference/ExecWait)的具体用法：

  ```nsis
  ExecWait command [user_var(exit code)]
  ```

## Reference

- https://nsis.sourceforge.io/Reference/CreateShortCut
- https://nsis.sourceforge.io/Reference/ExecWait

