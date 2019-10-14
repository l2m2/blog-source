---
title: Qt元对象系统
description: 主要翻译自官方文档。
tags: Qt
toc: false
date: 2019-10-14 13:57:00
---
Qt的元对象系统提供了信号与槽机制，运行时的类型信息，以及动态属性系统。

使用元对象系统的三个条件：

1. 只有继承自QObject的派生类才能享受元对象系统的好处。
2. 在类声明时添加Q_OBJECT宏开启元对象系统的功能。
3. 元对象编译器编译出moc_*.cpp, 帮助Object派生类实现必要的代码。

`moc`工具读取C++源文件，如果发现有类包含Q_OBJECT宏，它就创建另一个C++源文件（moc_*.cpp）,为每个类生成生成包含元对象实现的代码。生成的moc源文件通常被包含到类的源文件中，或者类的实现一同被编译和链接。

除了提供信号与槽机制以为，元对象系统还提供以下特性：

- `QObject::metaObject()` 返回该类相关的元对象。

- `QMetaObject::className()`返回运行时的类名，而无需编译器的RTTI(Run-Time Type Information)支持。【标准C++中通过 dynamic_cast和typeid 提供 RTTI机制。】

- `QObject::inherits()` 通过此函数判断某对象是否是QObject继承树上的实例。

  ```c++
  QTimer *timer = new QTimer;         // QTimer inherits QObject
  timer->inherits("QTimer");          // returns true
  timer->inherits("QObject");         // returns true
  timer->inherits("QAbstractButton"); // returns false
  ```

- `QObject::tr()`提供i18n支持。

- `QObject::setProperty()`和` QObject::property()`提供动态的属性系统， 可通过名称获取和设置对象属性。

- `QMetaObject::newInstance()`构建某个类的实例

除此之外还可以通过`qobject_cast()`进行QObject对象之前的动态转换，表现得就像标准C++中的`dynamic_cast`一样，但好处是你并不需要RTTI支持。【 C++虽然在后期定义了dynamic_cast和typeid两个关键字，但并没有说明如何实现这两个关键字。这就造成了不同的编译器的实现不同，更别说提供RTTI功能的库千差万别。由此导致的最大问题就是程序的可移植性差，项目之间无法完美兼容。】

例如，我们假定`MyWidget`继承自`QWidget`并且也声明了Q_OBJECT宏：

```c++
QObject *obj = new MyWidget;
```

类型为`QObject *`的`obj`变量，实际上指向一个`MyWidget`对象，因此我们可以这样转换：

```c++
QWidget *widget = qobject_cast<QWidget *>(obj);
```

上面的例子中，`QObject`转换成`QWidget`成功了，因为这个对象实际是一个`MyWidget`, 而`MyWidget`是`QWidget`的派生类。同样地，我们可以进行如下转换：

```c++
MyWidget *myWidget = qobject_cast<MyWidget *>(obj);
```

但是如果我们将它转换成`QLabel`呢，当然，会失败。

```
QLabel *label = qobject_cast<QLabel *>(obj);
// label is nullptr
```

所以我们可以通过`qobject_cast`来进行运行时的类型判断，例如：

```c++
if (QLabel *label = qobject_cast<QLabel *>(obj)) {
    label->setText(tr("Ping"));
} else if (QPushButton *button = qobject_cast<QPushButton *>(obj)) {
    button->setText(tr("Pong!"));
}
```

当然，我们可以在不用Q_OBJECT宏和元对象信息的情况下仍旧使用`QObject`作为基类，但是像信号和槽以及其他这里描述的特性将无法使用。以及，`QMetaObject::className()`不会返回真实类的名称。

**因此，不管你用不用信号与槽、属性，我们都强烈推荐所有QObject的派生类都使用`Q_OBJECT`宏。**

## 参考

- [The Meta-Object System]( https://doc.qt.io/archives/qt-4.8/metaobjects.html )
- [Qt中的元对象系统（Meta-Object System）]( https://zhuanlan.zhihu.com/p/61303678 )