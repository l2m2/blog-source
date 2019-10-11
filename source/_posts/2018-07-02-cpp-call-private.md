---
title: C++访问私有函数
toc: true
description: 用贱招访问私有函数，你值得拥有
date: 2018/7/2 00:00:00
tags:
- C++
---
## 前情提要

当然，本文不讨论基本的友元破封装的套路。

## 访问的私有函数是一个虚函数

虚函数可以通过虚表的地址算出来。

示例：

```c++
#include <QCoreApplication>
#include <iostream>

class Test {
private:
    virtual void func() {
        std::cout << __func__ << std::endl;
    }
    virtual void func2() {
        std::cout << __func__ << std::endl;
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Test t;
    unsigned int v_table_addr = *((unsigned int *) &t);
    typedef void (*pFunc)(void *);
    pFunc func = (pFunc)(*((unsigned int *)v_table_addr));
    func(&t);
    pFunc func2 = (pFunc)(*((unsigned int *)(v_table_addr + 4)));
    func2(&t);

    return a.exec();
}

```

## 访问的私有函数是普通函数

```c++
/// Ref:
/// http://bloglitb.blogspot.com/2010/07/access-to-private-members-thats-easy.html

#include <iostream>

template<typename Tag>
struct result {
    /* export it ... */
    typedef typename Tag::type type;
    static type ptr;
};
template<typename Tag>
typename result<Tag>::type result<Tag>::ptr;
template<typename Tag, typename Tag::type p>
struct rob : result<Tag> {
    /* fill it ... */
    struct filler {
        filler() { result<Tag>::ptr = p; }
    };
    static filler filler_obj;
};
template<typename Tag, typename Tag::type p>
typename rob<Tag, p>::filler rob<Tag, p>::filler_obj;

class A {
private:
    void func() {
        std::cout << __func__ << std::endl;
    }
    void func2(int n) {
        std::cout << __func__ << n << std::endl;
    }
};

struct Af { typedef void(A::*type)(); };
template class rob<Af, &A::func>;

struct Af2 { typedef void(A::*type)(int); };
template class rob<Af2, &A::func2>;

int main()
{
    A a;
    (a.*result<Af>::ptr)();
    (a.*result<Af2>::ptr)(4);
    return 0;
}

```

## 在Qt中访问私有槽函数

基于Qt的元对象系统，可以通过QMetaObject::invokeMethod唤起。

```C++
#include <QCoreApplication>
#include <iostream>

class Test: public QObject {
    Q_OBJECT
public:
    Test(QObject *parent = nullptr) : QObject(parent) {}
private slots:
    void func() {
        std::cout << __func__ << std::endl;
    }
};

#include "main.moc"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Test t;
    QMetaObject::invokeMethod(&t, "func");

    return a.exec();
}

```

