---
title: 解决“Could not install Gradle distribution from ...”的错误
toc: false
date: 2021-06-09 15:27:00
description: ...
tags:
- Android
---

在Windows 10 上安装完Android Studio后, 按照[官方指南](https://developer.android.google.cn/training/basics/firstapp/creating-project)创建工程后，出现如下错误：

```
Could not install Gradle distribution from 'https://services.gradle.org/distributions/gradle-6.7.1-bin.zip'.
```

解决方案：

1. 根据提示手动下载错误信息中的https://services.gradle.org/distributions/gradle-6.7.1-bin.zip

2. 进入`%USERPROFILE%\.gradle\wrapper\dists\gradle-6.7.1-bin\bwlcbys1h7rz3272sye1xwiv6`目录下。随机字符串`bwlcbys1h7rz3272sye1xwiv6`可能有差异。

   删除该目录下原来的所有文件。

3. 将步骤1中下载好的zip包解压到`%USERPROFILE%\.gradle\wrapper\dists\gradle-6.7.1-bin\bwlcbys1h7rz3272sye1xwiv6`下。

4. 重启Android Studio。

