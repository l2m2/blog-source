---
title: Python函数参数
toc: false
date: 2020-04-03 16:05:00
description: 默认参数，关键词参数，可变参数，指定类型
tags:
- Python
---

## 位置参数

先看两个普通的函数参数

```python
def foo():
  return 1
def bar(a, b):
  return a + b
r = foo()
print(f'r={r}')
s = bar(3, 4)
print(f's={s}')
```

foo函数没有任何参数， bar函数接受两个参数

如果给foo函数传参数会怎样，比如`foo(5, "str")`？

```
TypeError: foo() takes 0 positional arguments but 2 were given
```

同样地，给bar函数传大于或小于两个参数，也会报同样的错。

**因此，如果限定了函数签名，则必须按照指定的签名传入参数。**

如果给bar传入两个并不能相加的参数会怎样，比如`bar(5, "str")`?

```
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

## 限定类型

那么，是否可以像静态语言C++一样限定类型呢？

从Python 3.5 开始，我们可以像下面这样限定类型。

```python
def greeting(name: str) -> str:
  return 'Hello ' + name
```

除了基本的类型外，还可以限定类型为函数指针，泛型，自定义类型等等。具体玩法在[这里](https://docs.python.org/3/library/typing.html)。

## 默认参数

我们也可以给函数参数赋默认值。例如：

```python
def bar(a: int, b: int = 5) -> int:
  return a + b
r = bar(5)
print(r) # 10
```

那我们可以定义成`def bar(a: int = 5, b: int) ->int`吗？

```
SyntaxError: non-default argument follows default argument
```

不能。

**任何参数都可以由默认值，但是一旦有了默认参数，其右边的所有参数也必须具有默认值。**

## 关键词参数

Python允许使用关键字参数调用函数。例如：

```python
def bar(a: int, b: int) -> int:
  return a + b
r = bar(a=6, b=9)
print(r)
# 15
```

但在关键字参数之后放置位置参数将导致错误。例如：

```python
r = bar(a=6, 9)
print(r)
# SyntaxError: positional argument follows keyword argument
```

有没办法限制必须用关键词参数呢？有的。例如：

```python
def bar(a: int, *, b: int) -> int:
  return a + b
r = bar(6, b=9) 
# 此处改成 bar(6, 9) 会报错：
# TypeError: bar() takes 1 positional argument but 2 were given
print(r)
```

## 可变参数

可否像其他语言一样玩可变参数呢？可以！可变参数可以是位置参数，也可以是关键词参数。例如我们常在开源项目中看到这样的定义：

```python
def foo(*args, **kwargs):
  print(args, type(args))   
  print(kwargs, type(kwargs))
foo(4, 5, "str", a=5, b=6)
# (4, 5, 'str') <class 'tuple'>
# {'a': 5, 'b': 6} <class 'dict'>
```

上面的例子中，args收集所有的可变位置参数为一个元组，kwargs收集所有的关键词参数为一个字典。

需要注意的是，关键词参数不能写在位置参数前面。

## Reference

- https://www.programiz.com/python-programming/function-argument
- https://docs.python.org/3/library/typing.html
- https://juejin.im/post/5e785405f265da57455b692e