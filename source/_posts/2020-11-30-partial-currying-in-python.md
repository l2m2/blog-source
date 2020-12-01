---
title: Python3中的柯里化和偏函数
toc: false
date: 2020-11-24 10:59:00
description: ...
tags:
- Python
---

之前freecodecomp的时候有学到过[JavaScript中的柯里化和偏函数](/2019/07/08/2019-07-08-currying-partial-in-js/), 我们再一起对应看看Python3中说怎么玩的。

## 柯里化

柯里化是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

```python
def un_curried(x, y):
    return x + y

def curried(x):
    def tmp(y):
        return x + y
    return tmp

print(un_curried(2, 3))
# 5
print(curried(2)(3))
# 5
```



## 偏函数

偏函数是把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。

`functools.partial(func, /, *args, **keywords)`

```python
from functools import partial

def impartial(x, y, z):
    return x + y + z

partial_fn_2 = partial(impartial, 2)
print(partial_fn_2(3, 4)) 
# 9

partial_fn_2_3 = partial(impartial, 2, 3)
print(partial_fn_2_3(4))
# 9

partial_fn_null = partial(impartial)
print(impartial(2, 3, 4))
# 9
```

## Reference

- https://docs.python.org/zh-cn/3/library/functools.html

