---
title: QMap迭代Key的顺序
toc: true
date: 2020-10-13 10:19:00
description: 介绍QMap迭代Key的顺序，以及如何定制顺序
tags:
- Qt
---

## QMap

QMap是Qt应用程序开发中最常用的数据结构了，那么它是怎么存储的，迭代出来的顺序又是根据什么确定的呢？

先看官方文档，

The [QMap](qmap.html) class is a template class that provides a red-black-tree-based dictionary.

[QMap](qmap.html)<Key, T> is one of Qt's generic [container classes](containers.html). It stores (key, value) pairs and provides fast lookup of the value associated with a key.

[QMap](qmap.html) and [QHash](qhash.html#qhash) provide very similar functionality. The differences are:

- [QHash](qhash.html#qhash) provides average faster lookups than [QMap](qmap.html). (See [Algorithmic Complexity](containers.html#algorithmic-complexity) for details.)
- When iterating over a [QHash](qhash.html#qhash), the items are arbitrarily ordered. With [QMap](qmap.html), the items are always sorted by key.
- The key type of a [QHash](qhash.html#qhash) must provide operator==() and a global [qHash](qhash.html#qhash)(Key) function. The key type of a [QMap](qmap.html) must provide operator<() specifying a total order.

从上面的官方文档中我们得知，QMap是基于[红黑树](https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91)实现的，所以存储没有啥顺序可言。当我们迭代QMap时，是按照Key的顺序来的。

## Key : int

我们来看几个基础的例子，

```c++
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<int, QString> map1;
    map1.insert(2, QString("value2"));
    map1.insert(1, QString("value1"));
    map1.insert(11, QString("value11"));
    map1.insert(5, QString("value5"));

    for (QMap<int, QString>::const_iterator it = map1.constBegin(); it != map1.constEnd(); it++) {
        qDebug() << it.key() << it.value();
    }

    return a.exec();
}
```

输出结果是，

```
1 "value1"
2 "value2"
5 "value5"
11 "value11"
```

## Key : QString

没问题，我们把key换成QString类型，输出会是什么呢？

```c++
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<QString, QString> map1;
    map1.insert(QString("2"), QString("value2"));
    map1.insert(QString("1"), QString("value1"));
    map1.insert(QString("11"), QString("value11"));
    map1.insert(QString("5"), QString("value5"));

    for (QMap<QString, QString>::const_iterator it = map1.constBegin(); it != map1.constEnd(); it++) {
        qDebug() << it.key() << it.value();
    }

    return a.exec();
}
```

没错，结果是按照字符串的顺序来的。

```
"1" "value1"
"11" "value11"
"2" "value2"
"5" "value5"
```

## Key : struct

```c++
struct Person {
    quint8 age;
    QString name;
};

inline bool operator < (const Person &left, const Person &right) {
    return left.age < right.age;
}

inline QDebug operator<< (QDebug d, const Person &model) {
    d << "Name: " << model.name << ",Age: " << model.age;
    return d;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<Person, QString> map1;
    map1.insert(Person{34, "Leon"}, QString("leon"));
    map1.insert(Person{24, "Jim"}, QString("jim"));
    map1.insert(Person{45, "Lin Tao"}, QString("lintao"));
    map1.insert(Person{28, "Han meimei"}, QString("hanmeimei"));

    for (QMap<Person, QString>::const_iterator it = map1.constBegin(); it != map1.constEnd(); it++) {
        qDebug() << it.key() << it.value();
    }

    return a.exec();
}
```

当Key是一个结构体时，它的顺序是啥呢？

由于我们重载了Person的operator < 函数，所以结果是按照年龄排序。

```
Name:  "Jim" ,Age:  24 "jim"
Name:  "Han meimei" ,Age:  28 "hanmeimei"
Name:  "Leon" ,Age:  34 "leon"
Name:  "Lin Tao" ,Age:  45 "lintao"
```

**如果我们注释掉operator < 部分的代码，会发生什么呢？**

会产生一个编译错误：

```
C:\Qt\Qt5.6.3\5.6.3\msvc2015\include\QtCore\qmap.h:68: error: C2678: binary '<': no operator found which takes a left-hand operand of type 'const Person' (or there is no acceptable conversion)
```

为啥呢？

看看QMap的源码，我们知道，QMap比较Key的顺序时，是根据qMapLessThanKey函数的执行结果来判断的，而qMapLessThanKey的实现是必须要求Person有重载operator < 的。

```c++
template <class Key> inline bool qMapLessThanKey(const Key &key1, const Key &key2)
{
    return key1 < key2;
}

template <class Ptr> inline bool qMapLessThanKey(const Ptr *key1, const Ptr *key2)
{
    Q_STATIC_ASSERT(sizeof(quintptr) == sizeof(const Ptr *));
    return quintptr(key1) < quintptr(key2);
}
```

## Key : pointer

从上面的qMapLessThanKey部分的代码已经可以看出，当Key是一个指针时，是根据内存地址的整数值排序的。

```c++
struct Person {
    quint8 age;
    QString name;
};

inline QDebug operator<< (QDebug d, const Person *model) {
    d << "Name: " << model->name << ",Age: " << model->age;
    return d;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<Person *, QString> map1;
    map1.insert(new Person{34, "Leon"}, QString("leon"));
    map1.insert(new Person{24, "Jim"}, QString("jim"));
    map1.insert(new Person{45, "Lin Tao"}, QString("lintao"));
    map1.insert(new Person{28, "Han meimei"}, QString("hanmeimei"));

    for (QMap<Person *, QString>::const_iterator it = map1.constBegin(); it != map1.constEnd(); it++) {
        qDebug() << it.key() << it.value();
    }

    return a.exec();
}
```

输出：

```
Name:  "Leon" ,Age:  34 "leon"
Name:  "Jim" ,Age:  24 "jim"
Name:  "Lin Tao" ,Age:  45 "lintao"
Name:  "Han meimei" ,Age:  28 "hanmeimei"
```

这是不是make no sence！所以一般情况下，我们不用pointer当QMap的Key, 若确实需要，我们用QHash替换它（既然顺序没有意义，不如要一个更好的查找效率）。

**那如果硬要使用Pointer为Key，又要排序咋整？**

对qMapLessThanKey进行特化：

```c++
struct Person {
    quint8 age;
    QString name;
};

inline QDebug operator<< (QDebug d, const Person *model) {
    d << "Name: " << model->name << ",Age: " << model->age;
    return d;
}

template<> inline bool qMapLessThanKey<Person *>(Person * const & key1, Person * const & key2)
{
    return key1->age < key2->age;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<Person *, QString> map1;
    map1.insert(new Person{34, "Leon"}, QString("leon"));
    map1.insert(new Person{24, "Jim"}, QString("jim"));
    map1.insert(new Person{45, "Lin Tao"}, QString("lintao"));
    map1.insert(new Person{28, "Han meimei"}, QString("hanmeimei"));

    for (QMap<Person *, QString>::const_iterator it = map1.constBegin(); it != map1.constEnd(); it++) {
        qDebug() << it.key() << it.value();
    }

    return a.exec();
}

```

这样结果就会按照年龄的升序打印了：

```
Name:  "Jim" ,Age:  24 "jim"
Name:  "Han meimei" ,Age:  28 "hanmeimei"
Name:  "Leon" ,Age:  34 "leon"
Name:  "Lin Tao" ,Age:  45 "lintao"
```

## std::map

标准库实现的Map提供了自定义key排序的功能，下面是一个例子。

```c++
struct dirty {
    bool operator()(const int & a, const int & b) const {
        return a > b;
    }
};

int main(int argc, char *argv[])
{
    std::map<int, std::string, dirty> map1;
    map1[2] = "value2";
    map1[1] = "value1";
    map1[11] = "value11";
    map1[5] = "value5";

    for (std::map<int, std::string>::const_iterator it = map1.cbegin(); it != map1.cend(); it++) {
        std::cout << it->first << " " << it->second << std::endl;
    }

    return 0;
}

```

它的输出是，

```
11 value11
5 value5
2 value2
1 value1
```

## Reference

- https://stackoverflow.com/questions/2677577/how-to-overload-operator-for-qdebug
- https://github.com/KDE/clazy/blob/master/docs/checks/README-qmap-with-pointer-key.md

- https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91

