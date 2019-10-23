---
title: Qt中用qobject_cast<>进行RTTI时需注意
toc: false
date: 2019-10-23 17:02:21
description: 当有多重继承时，可能产生意想不到的问题
tags: 
- Qt
---
例子：

```c++
#include <QCoreApplication>
#include <QDebug>
#include <typeinfo>
class Base : public QObject
{
    Q_OBJECT
public:
    explicit Base(QObject *parent = 0) {}
};
class Foo : public Base
{
    Q_OBJECT
public:
    Foo(QObject *parent = 0) : Base(parent) {}
};
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    Base *obj = new Foo();
    auto b = qobject_cast<Base *>(obj);
    qDebug() << (b != nullptr);
    qDebug() << obj->metaObject()->className();
    qDebug() << typeid(obj).name();
    return a.exec();
}
#include "main.moc"
```

输出结果：
```
true
Foo
class Base *
```

不难看出，当需要明确根据类型进行不同逻辑时，可能会产生意想不到的问题。

在我们的项目中，有一个表单类TUiLoader。有两个控件类，分别是TLineEdit和TDateTimeEdit。
大概长这样：

```c++
class TLineEdit {};
class TDateTimeEdit : public TLineEdit {};
class TUiLoader : public QFrame {
    Q_OBJECT
public:
	void doSomething(QObject *obj);    
private:
    QList<QObject *> objs;
}
```

当我们在`doSomething`函数中要根据不同的类型进行不同逻辑时，例如：

```c++
void TUiLoader::doSomething(QObject *obj) {
    if (qobject_cast<TLineEdit *>(obj)) {
        // do TLineEdit thing
    } else if (qobject_cast<TDateTimeEdit *>(obj)) {
        // do TDateTimeEdit thing
    }
}
```

危险！TDateTimeEdit当然可以转换成TLineEdit, 因此执行了TLineEdit的逻辑，这并不是我想要的。

如何解决？

可以在`doSomething`中调换判断的顺序，或者用元对象的className进行RTTI。