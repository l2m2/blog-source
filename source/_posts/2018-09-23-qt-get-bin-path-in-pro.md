---
title: 在pro中获取Qt的安装路径
description: 假如你需要在qmake时执行qt的一些程序，可能用得上
date: 2018-09-23 00:00:00
tags:
- Qt
---
- 方法一
  ```
  TEMPNAME = $${QMAKE_QMAKE}
  QTPATH = $$dirname(TEMPNAME)
  message($$QTPATH)
  ```
- 方法二
  ```
  QTPATH = $$[QT_INSTALL_LIBS]/../bin
  message($$QTPATH)
  ```

