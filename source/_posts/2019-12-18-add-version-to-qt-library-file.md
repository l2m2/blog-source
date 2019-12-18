---
title: QMAKE为DLL添加版本信息
toc: false
date: 2019-12-18 15:13:07
description: 在PRO文件中添加生成DLL文件的版本信息
tags:
- Qt
---

在PRO文件中添加如下代码：

```
win32 {
    CONFIG += skip_target_version_ext
    VERSION = 1.0.0.0
    QMAKE_TARGET_PRODUCT = XXX
    QMAKE_TARGET_COMPANY = XXX Co.,Ltd.
    QMAKE_TARGET_COPYRIGHT = Copyright(C) 2019 XXX
}
```

`CONFIG += skip_target_version_ext`表示生成的dll文件中不添加版本的后缀。

效果如下：
![](../images/add-version-to-library-file.png)