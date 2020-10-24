---
title: Qt中的事件循环
toc: false
date: 2020-10-24 13:37:00
description: 东拼西凑，没啥干货
tags:
- Qt
---

我们先看一个程序，

```c++
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow w;
    w.show();
    return a.exec();
}
```

很熟悉对不对，每一个初学Qt的同学都见过。

`QApplication`是个啥？`a.exec()`在干嘛？

## 事件

和之前GUI框架一样，Qt也是事件驱动的。

这里的“事件”可能来自应用程序自己，也可能来自外部。

比如用户交互：鼠标单击或按键，或与其他线程通信，也可能来自操作系统或其他进程的消息。

Qt把这些事件封装成了`QEvent`, 比如`QKeyEvent`, `QMouseEvent`. 当然，你也可以自定义自己的事件。

## 事件循环

事件循环做些啥？它看起来像下面这样：

```c++
while (!interrupted) {
    if (there are new events) {
        dispatch them
    }
    wait for more events
}
```

![](/images/qt-event-loop-1.jpg)

Qt中对应的事件循环的类是`QEventLoop`.

```c++
class QEventLoop : public QObject {
public:
    bool processEvents(ProcessEventsFlags flags = AllEvents);
    void processEvents(ProcessEventsFlags flags, int maximumTime);
	int exec(ProcessEventsFlags flags = AllEvents);
    void exit(int returnCode = 0);
};
```

文章开头的`a.exec()` 实际上是开启了一个事件循环。

```c++
int QCoreApplication::exec()
{
    // ...
    QEventLoop eventLoop;
    self->d_func()->in_exec = true;
    self->d_func()->aboutToQuitEmitted = false;
    int returnCode = eventLoop.exec();
    //...
    return returnCode;
}
```

所有的事件分发、事件处理都从`a.exec()`开始。

而当我们调用`qApp->quit()`退出应用程序时，实际上也退出了事件循环。

```c++
void QCoreApplication::exit(int returnCode)
{
	// ...
    for (int i = 0; i < data->eventLoops.size(); ++i) {
        QEventLoop *eventLoop = data->eventLoops.at(i);
        eventLoop->exit(returnCode);
    }
    // ...
}
```

### 程序未响应？

![](/images/qt-event-loop-2.png)

程序在什么情况下会出现未响应的情况？

当我们在主线程中进行耗时操作，阻塞了事件循环，无法响应其他事件时，就会出现类似上面的假死现象。

如何避免？

一般地，我们会在另外的线程中处理耗时的操作。

那除了另开线程，还有没有其他方式？

### `processEvents()`

有，就是强制调用`processEvents()`

```c++
for (auto i : list) {
    // do something
    QCoreApplication::processEvents();
}
```

虽然这也能避免程序假死，但通常我们不建议这么做。因为它很难确定应多久调用一次`processEvents()`来最大化可用性和性能，这取决于硬件。

### 另起事件循环

还有，我们可以通过另起一个事件循环来避免程序假死。我们通常在访问网络资源时这么做。

```c++
QNetworkReply *reply = manager->post(request, body);
QEventLoop loop;
connect(reply, SIGNAL(finished()), &loop, SLOT(quit()));
connect(reply, SIGNAL(error(QNetworkReply::NetworkError)), &loop, SLOT(quit()));
loop.exec();
```

通过另起事件循环，我们把本身异步的操作变成了同步，在实际编码过程中，这很有效。

## Reference

- https://zhuanlan.zhihu.com/p/72758194
- https://www.cleanqt.io/blog/crash-course-in-qt-for-c%2B%2B-developers,-part-1
- https://www.qtdeveloperdays.com/2013/sites/default/files/presentation_pdf/Qt_Event_Loop.pdf

