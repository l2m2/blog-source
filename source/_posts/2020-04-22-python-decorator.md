---
title: Python - 装饰器
toc: false
date: 2020-04-21 22:25:00
description: 装饰器是Python重要的特性。
tags:
- Python
---

装饰器，顾名思义，就是给本身的对象加以装饰。动态的将额外的功能添加到已有的对象上。

可以理解成给礼物打包，先将礼物放在盒子里，然后包装盒子。

```python
def box(func):
    def _inner():
        print("in box")
        func()
    return _inner

def gift():
    print("I am gift")

gift = box(gift)
gift()
```

执行结果：

```
in box
I am gift
```

Python在语法上对装饰器进行了支持。

下面的代码和上面的代码等价。

```python
def box(func):
    def _inner():
        print("in box")
        func()
    return _inner

@box
def gift():
    print("I am gift")

gift()
```

当然，也可以加多个装饰器。例如：

```python
def star(func):
    def inner(*args, **kwargs):
        print("*" * 30)
        func(*args, **kwargs)
        print("*" * 30)
    return inner

def percent(func):
    def inner(*args, **kwargs):
        print("%" * 30)
        func(*args, **kwargs)
        print("%" * 30)
    return inner

@star
@percent
def printer(msg):
    print(msg)

printer("Hello")
```

执行结果：

```
******************************
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Hello
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
******************************
```

更多装饰器的用法，[这篇文章](https://realpython.com/primer-on-python-decorators/)写的很详细。

## Reference

- https://www.programiz.com/python-programming/decorator

- https://realpython.com/primer-on-python-decorators/