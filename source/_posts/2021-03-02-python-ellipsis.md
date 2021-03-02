---
title: Python - 省略号
toc: false
date: 2021-03-02 16:05:00
description: Ellipsis
tags:
- Python
---

## Ellipsis

在Python中，一切皆对象，`...`是Ellipsis对象。

```shell
>>> ...
Ellipsis
>>> type(...)
<class 'ellipsis'>
```

它是一个单例。

```shell
>>> id(...)
4534353712
>>> id(...)
4534353712
```

## 作用

### numpy中的切片操作

```python
import numpy as np

array = np.random.rand(2, 2, 2, 2)
print(array[..., 0])
```

在上面的例子中，`[:, :, :, 0]` , `[Ellipsis, 0]`和`[..., 0]`是等价的。

### 类型提示

当函数中的参数允许任何类型时

下面的例子假定x的类型为Any

```python
def foo(x: ...) -> None:
  return x + x

print(foo(2))
print(foo("e"))
```

下面的例子假定func的参数为Any, 返回值为int

```python
from typing import Callable

def inject(func: Callable[..., int]) -> None:
  return func() + 5

print(inject(lambda: 4))
```

### 当成pass用

```python
def foo():
  ...
```

### 当成参数默认值用

```python
def foo(x=...):
  return x

print(foo()) # output: Ellipsis
print(foo(2)) # output: 2

def bar(x=None):
  return x

print(bar()) # output: None
print(bar(2)) # output: 2
```

## Reference

- https://www.geeksforgeeks.org/what-is-three-dots-or-ellipsis-in-python3/