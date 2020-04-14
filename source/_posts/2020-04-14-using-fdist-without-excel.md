---
title: Qt中调用Excel中的函数
toc: false
date: 2020-04-14 16:54:00
description: QAxObject不打开Excel，直接调
tags:
- Qt
---

今天遇到一个业务场景，需要在Qt中调用Excel中的[FDIST函数](https://support.office.com/en-us/article/fdist-function-ecf76fba-b3f1-4e7d-a57e-6a5b7460b786)。

此函数返回两个数据集的（右尾）F 概率分布（变化程度）。 使用此函数可以确定两组数据是否存在变化程度上的不同。 例如，分析进入中学的男生、女生的考试分数，来确定女生分数的变化程度是否与男生不同。

用Qt去实现太麻烦，直接用Excel中的函数好了。代码如下：

```c++
#include <QCoreApplication>
#ifdef Q_OS_WIN
#include <objbase.h>
#include <QAxObject>
#endif
#include <QDebug>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

#ifdef Q_OS_WIN
    ::CoInitialize(NULL);
    QAxObject *obj = new QAxObject("Excel.Application");
    double x = 7.25284;
    int deg_freedom1 = 8;
    int deg_freedom2 = 3;
    QString expression = QString("Evaluate(\"FDIST(%1, %2, %3)\")")
            .arg(x).arg(deg_freedom1).arg(deg_freedom2);
    auto result = obj->dynamicCall(expression.toStdString().c_str());
    qDebug() << result;
#endif

    return a.exec();
}
```

## Reference

- https://support.office.com/en-us/article/fdist-function-ecf76fba-b3f1-4e7d-a57e-6a5b7460b786
- https://www.get-digital-help.com/how-to-use-the-fdist-function/
- https://forum.qt.io/topic/106836/how-to-run-vba-function-using-qaxobject/5