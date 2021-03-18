---
title: 在Python中使用正则
toc: true
date: 2020-11-21 18:38:00
description: 汇总下常用的用法，打花这无聊的周末。
tags:
- Python
- 正则
---

这篇文章假定你已经熟知了正则的基础规则。

## re

Python中有一个正则相关的模块，`re`.

你只需要引入该模块即可。

```python
import re
```

## re.findall()

`re.findall(pattern, string, flags=0)`

返回所有匹配的字符串列表。

```python
print(re.findall(r'\d+', 'Hello 12 hi 89. Howdy 34'))
# ['12', '89', '34']
```

如果包含group呢？

```python
print(re.findall(r'Hello\s*(\d+)', 'Hello 12 hi 89. Howdy 34'))
# ['12']
```

如果包含多个group呢？

```python
print(re.findall(r'Hello\s(\d+)\shi\s(\d+)', 'Hello 12 hi 89. Howdy 34'))
# [('12', '89')] # 返回元组的列表
print(re.findall(r'Hello\s*(\d+)|S*(\d+)', 'Hello 12 hi 89. Howdy 34'))
# [('12', ''), ('', '89'), ('', '34')] # 包含空匹配
```

## re.split()

`re.split(pattern, string, maxsplit=0, flags=0)`

分割字符串，返回分割出来的列表。

```python
print(re.split(r'\s+', 'Vue is a progressive framework for building user interfaces.'))
# ['Vue', 'is', 'a', 'progressive', 'framework', 'for', 'building', 'user', 'interfaces.']
print(re.split(r'\s+', 'Vue is a progressive framework for building user interfaces.', 3))
# ['Vue', 'is', 'a', 'progressive framework for building user interfaces.'] # 最多分割三次
```

如果没有匹配呢？

```python
print(re.split(r'\d+', 'Vue is a progressive framework for building user interfaces.'))
# ['Vue is a progressive framework for building user interfaces.'] # 没有匹配返回包含原字符串的列表
```

不区分大小写

```python
print(re.split('[a-f]+', '0a3B9', flags=re.IGNORECASE))
# ['0', '3', '9']
```

## re.sub()

`re.sub(pattern, replace, string, count=0, flags=0)`

返回用`replace`替换`pattern`匹配的字符串。

```python
print(re.sub(r'\s+', '', 'abc 12 \n r \t'))
# abc12r
print(re.sub(r'BUG#(\d+)', r'BUG[#\1](http://zentao.xxx.net/zentao/bug-view-\1.html)', '修复Bug#4354', flags=re.I))
# 修复BUG[#4354](http://zentao.xxx.net/zentao/bug-view-4354.html)
```

如果没有匹配，将返回原字符串

```python
print(re.sub(r'\d+', '', 'abc de'))
# abc de
```

只替换前2个

```python
print(re.sub(r'\s+', '', 'abc de f  g', count=2))
# abcdef  g
```

使用`re.subn()`会返回替换后的字符串和替换的个数。

```python
print(re.subn(r'\s+', '', 'abc de f  g'))
# ('abcdefg', 3)
```

一个稍微复杂一点的例子，将`/users/{a1}/code/{b1}/ff`字符串中大括号包裹的部分替换为大括号中的内容再加一个后缀suffix

```python
re.sub(r'\{(.*?)\}', r'\1suffix', r'/users/{a1}/code/{b1}/ff')
# /users/a1suffix/code/b1suffix/ff
```



## re.search()

`re.search(pattern, string, flags=0)`

扫描字符串以查找正则匹配的第一个位置，然后返回相应的匹配对象。 如果没有匹配返回None。

```python
print(re.match(r'\d+', 'abc12de345'))
# None
print(re.match(r'.*?\d+', 'abc12de345'))
# <re.Match object; span=(0, 5), match='abc12'>
```





## Reference

- https://www.programiz.com/python-programming/regex

- https://docs.python.org/3/library/re.html

  