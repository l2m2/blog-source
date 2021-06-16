---
title: JRE 8裁剪
toc: false
date: 2021-06-16 14:16:00
description: Java 8
tags:
- Java
---

基于Java 8 的裁剪。

```shell
$ java.exe -version
java version "1.8.0_291"
Java(TM) SE Runtime Environment (build 1.8.0_291-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.291-b10, mixed mode)
```

## 删除多余的文件

...

## 重新压缩jar

当前进行裁剪版本的rt.jar达到了61M，我们来对它进行重新压缩

```shell
$ ls rt.jar -lh
-rw-r--r-- 1 GW00235770 1049089 61M Jan 20 16:30 rt.jar
```

我们先对它解压

```shell
$ mkdir rtjar
$ cd rtjar
$ jar -xf ../rt.jar
```

再重新压缩

```shell
$ zip -q -r rt.jar .
```

重新压缩后的大小, 32M

```shell
$ ls rt.jar -lh
-rw-r--r-- 1 GW00235770 1049089 32M Jun 16 14:28 rt.jar
```

我们可以用同样的方式对其他jar进行重新压缩

## Reference

- https://blogs.oracle.com/jtc/reducing-your-java-se-runtime-environment-footprint-legally