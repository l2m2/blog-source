---
title: 为什么Linux下Qt构建库文件生成多个so文件
toc: false
date: 2019-12-18 17:44:10
description: 我是Linux白痴，大佬们请无视
tags:
- Linux
- Qt
---

## 为了兼容性

[这篇文章](https://www.ibm.com/developerworks/cn/linux/l-shlibs.html)对SO（Shared Object）有更详尽的介绍。

**举例：**

我们有两个共享程序库DirtyA与DirtyB, 分别在pro中指定版本都为1.0.0.0。其中DirtyB依赖于DirtyA。

```
VERSION = 1.0.0.0
CONFIG += skip_target_version_ext # 生成的so名称上不会带上版本，否则生成libDirtyA1.so
```

DirtyA中提供接口foo:

```c++
int DirtyA::foo(int a, int b)
{
    return a + b;
}
```

DirtyB中提供接口foo:

```c++
int DirtyB::foo(int a, int b)
{
    DirtyA obj;
    int tmp = obj.foo(a, b);
    return tmp / 2;
}
```

那么生成的初始so文件有：

```
rwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyA.so -> libDirtyA.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyA.so.1 -> libDirtyA.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyA.so.1.0 -> libDirtyA.so.1.0.0
-rwxrwxr-x. 1 leon leon 27328 Dec 22 18:16 libDirtyA.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so -> libDirtyB.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so.1 -> libDirtyB.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so.1.0 -> libDirtyB.so.1.0.0
-rwxrwxr-x. 1 leon leon 27752 Dec 22 18:16 libDirtyB.so.1.0.0
```

除了libDirtyA.so.1.0.0和libDirtyB.so.1.0.0之外，都是软链接。

通过`ldd`( *list dynamic dependencies* ) 可以看到DirtyB依赖于DirtyA。

```
[leon@192 dist]$ ldd libDirtyB.so
	linux-vdso.so.1 =>  (0x00007fff69d99000)
	libDirtyA.so.1 (0x00007fa0c47ba000)
	libQt5Core.so.5 => /opt/Qt/Qt5.6.3/5.6.3/gcc_64/lib/libQt5Core.so.5 (0x00007fa0c40a0000)
```

编写一个Demo来调用接口测试：

```c++
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    DirtyB obj;
    int r = obj.foo(4, 5);
    qDebug() << r;
    return a.exec();
}
```

执行结果为4.

**结论1：链接时不会指定版本**

我们可以看到，在pro中添加依赖时，是这样写的：

```
unix:!macx: LIBS += -L$$PWD/../dist/ -lDirtyA
```

并不会指定版本。

 如果为链接器指定了 `-lDirtyA`，那么它会去寻找 `libDirtyA.so`，而`libDirtyA.so`本身是一个软链接，它指向的是libDirtyA.so.1.0.0。 

**接下来，我们修改DirtyA中foo的实现，并提升版本为1.1.0.0**，如下：

```c++
int DirtyA::foo(int a, int b)
{
    return a * b;
}
```

生成的目录下变成了这样：

```
[leon@192 dist]$ ll
total 84
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:31 libDirtyA.so -> libDirtyA.so.1.1.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:31 libDirtyA.so.1 -> libDirtyA.so.1.1.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyA.so.1.0 -> libDirtyA.so.1.0.0
-rwxrwxr-x. 1 leon leon 27328 Dec 22 18:16 libDirtyA.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:31 libDirtyA.so.1.1 -> libDirtyA.so.1.1.0
-rwxrwxr-x. 1 leon leon 27328 Dec 22 18:31 libDirtyA.so.1.1.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so -> libDirtyB.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so.1 -> libDirtyB.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so.1.0 -> libDirtyB.so.1.0.0
-rwxrwxr-x. 1 leon leon 27752 Dec 22 18:16 libDirtyB.so.1.0.0
```

libDirtyA.so指向了libDirtyA.so.1.1.0。

**构建DirtyC时不会重新编译**，运行结果是10.

**最后，我们继续修改DirtyA中foo的实现，但提升版本为2.0.0.0**，如下：

```c++
int DirtyA::foo(int a, int b)
{
    return a * b * 2;
}
```

生成的目录变成了这样：

```
[leon@192 dist]$ ll
total 112
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:36 libDirtyA.so -> libDirtyA.so.2.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:36 libDirtyA.so.1 -> libDirtyA.so.1.1.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyA.so.1.0 -> libDirtyA.so.1.0.0
-rwxrwxr-x. 1 leon leon 27328 Dec 22 18:16 libDirtyA.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:36 libDirtyA.so.1.1 -> libDirtyA.so.1.1.0
-rwxrwxr-x. 1 leon leon 27328 Dec 22 18:36 libDirtyA.so.1.1.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:36 libDirtyA.so.2 -> libDirtyA.so.2.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:36 libDirtyA.so.2.0 -> libDirtyA.so.2.0.0
-rwxrwxr-x. 1 leon leon 27328 Dec 22 18:36 libDirtyA.so.2.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so -> libDirtyB.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so.1 -> libDirtyB.so.1.0.0
lrwxrwxrwx. 1 leon leon    18 Dec 22 18:16 libDirtyB.so.1.0 -> libDirtyB.so.1.0.0
-rwxrwxr-x. 1 leon leon 27752 Dec 22 18:16 libDirtyB.so.1.0.0
```

我们不构建DirtyB和DirtyC, 直接运行DirtyC.运行结果仍然是10.

这是因为DirtyC和DirtyB仍然依赖的是DirtyA.so.1.

**结论2： 当加载DirtyC时，它将尝试去加载 `libDirtyA.so.1`， 其中 *1* 是它先前链接的版本 **

当然，如果重新构建DirtyB和DirtyC，得到的结果会变成20。而它们依赖的DirtyA主版本也变成了2。

## 如何不生成带numbers的so文件？

虽然生成带版本信息的so文件是更好的实践，但如何不生成呢？

只需要在pro中添加

```
CONFIG += plugin
```

生成的文件变成了这样：

```
[leon@192 dist]$ ll
total 312
-rwxrwxr-x. 1 leon leon 261320 Dec 22 18:51 DirtyC
-rwxrwxr-x. 1 leon leon  27328 Dec 22 18:50 libDirtyA.so
-rwxrwxr-x. 1 leon leon  27752 Dec 22 18:50 libDirtyB.so
```

**这样错的坏处是？**

没有版本信息，总是依赖最新的so，无兼容性可言。

## Reference

- [soname](https://en.wikipedia.org/wiki/Soname)
- [How to prevent generating multiple so files?](https://stackoverflow.com/questions/13295495/how-do-i-prevent-qt-creator-from-generating-multiple-so-files-for-the-same-libr)

- [Configuring qmake](https://doc.qt.io/qt-5/qmake-environment-reference.html)
- [What is the soname option for building shared libraries for ?](https://stackoverflow.com/questions/12637841/what-is-the-soname-option-for-building-shared-libraries-for)
- [How do so shared object numbers work?](https://unix.stackexchange.com/questions/475/how-do-so-shared-object-numbers-work)
- [剖析共享程序库](https://www.ibm.com/developerworks/cn/linux/l-shlibs.html)





