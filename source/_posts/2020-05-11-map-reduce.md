---
title: Map Reduce
toc: true
date: 2020-05-11 10:34:00
description: Map Reduce大法好！！！
tags:
- Python
- JavaScript
- C++
---

## 什么是Map/Reduce

维基百科中的解释：

> **MapReduce** is a [programming model](https://en.wikipedia.org/wiki/Programming_model) and an associated implementation for processing and generating [big data](https://en.wikipedia.org/wiki/Big_data) sets with a [parallel](https://en.wikipedia.org/wiki/Parallel_computing), [distributed](https://en.wikipedia.org/wiki/Distributed_computing) algorithm on a [cluster](https://en.wikipedia.org/wiki/Cluster_(computing)).

它是一种编程模型和相关的实现，用于在集群上使用并行分布式算法来处理和生成大数据集。

`map`将传入的函数依次作用到序列的每个元素，并把结果作为新的list返回。

`reduce`把一个函数作用在一个序列上, 并把结果继续和序列的下一个元素做累积计算。

## Python3中的Map/Reduce

### Map

```python
l = [2, 3, 4]
l = list(map(lambda x: x * 2, l))
print(l)
# [4, 6, 8]
```

将列表中的每个元素都乘以2. 

Map接受两个参数。

第一个参数是一个函数。这个函数接受一个参数，它就是迭代的元素。

第二个参数是被迭代的序列。

**注意**：在Python2中返回的是list， 而在Python3中返回的是一个`map object`.

### Reduce

```python
import functools
l = [2, 3, 4]
sum = functools.reduce(lambda prev, curr: prev + curr * 2, l, 0)
print(sum)
# 18
```

将列表中的每个元素乘以2再累加求和。

再来一个稍微更复杂的例子。

```python
import functools

def reduce_func(prev, curr):
    prev[curr["name"]] = curr["age"]
    return prev

l = [
    { "name": "A", "age": 12 },
    { "name": "B", "age": 20 },
    { "name": "C", "age": 16 },
    { "name": "D", "age": 30 },
    { "name": "E", "age": 56 },
]

result = functools.reduce(reduce_func, l, {})
print(result)
# {'A': 12, 'B': 20, 'C': 16, 'D: 30, 'E': 56}
```

Reduce接受三个参数。

第一个参数是一个函数。这个函数接受两个参数，第一个是上一个累加的值，第二个是当前迭代的元素。

第二个参数是被迭代的序列。

第三个参数是初始累加值。

## JavaScript中的Map/Reduce

用法和Python中的大同小异。

### Map

```js
let l = [2, 3, 4]
l = l.map( x => x * 2)
console.log(l)
// [ 4, 6, 8 ]
```

更详细的介绍参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map).

### Reduce

```js
let l = [2, 3, 4]
result = l.reduce((prev, curr) => prev + curr, 0)
console.log(result)
// 9
```

```js
let l = [
    { "name": "A", "age": 12 },
    { "name": "B", "age": 20 },
    { "name": "C", "age": 16 },
    { "name": "D", "age": 30 },
    { "name": "E", "age": 56 },
];

let result = l.reduce( (prev, curr) => { prev[curr.name] = curr.age; return prev;}, {} );
console.log(result); 
// { A: 12, B: 20, C: 16, D: 30, E: 56 }
```

更详细的介绍参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce).

**注意**：若JS解析引擎使用的是es6之前的标准，可用[lodash](https://lodash.com/)替代。

## C++中的Map/Reduce

### Map

```c++
#include <iostream>
#include <vector>
#include <algorithm>

int main(int argc, char *argv[])
{
    std::vector<int> v{2, 3, 4};
    std::transform(v.begin(), v.end(), v.begin(), [](int i) { return i * 2; });
    std::for_each(v.begin(), v.end(), [](int i) { std::cout << i << std::endl; });
    return 0;
}
// 4
// 6
// 8
```

上面这种方式会直接修改原序列。

如果我们不期望修改原序列，应该这样 ：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>

int main(int argc, char *argv[])
{
    std::vector<int> v{2, 3, 4};
    std::vector<int> ordinals;
    std::transform(v.begin(), v.end(), std::back_inserter(ordinals), [](int i) { return i * 2; });
    std::for_each(ordinals.begin(), ordinals.end(), [](int i) { std::cout << i << std::endl; });
    return 0;
}
```

### Reduce

直到C++17标准，才加入了`std::reduce`.

一个简单的替代实现：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>
#include <functional>

template <typename T>
T reduce(const std::vector<T>& data,
         const std::function<T(T,T)>& reduceFn) {
    typedef typename std::vector<T>::const_iterator Iterator;
    Iterator it = data.cbegin();
    Iterator end = data.cend();
    if (it == end) {
        throw 0;
    } else {
        T accumulator = *it;
        ++it;
        for (; it != end; ++it) {
            accumulator = reduceFn(accumulator, *it);
        }
        return accumulator;
    }
}

int main(int argc, char *argv[])
{
    std::vector<int> v{2, 3, 4};
    int result = reduce<int>(v, [](int prev, int curr){ return prev + curr; });
    std::cout << result << std::endl;
    return 0;
```

## Reference

- https://en.wikipedia.org/wiki/MapReduce
- https://www.liaoxuefeng.com/wiki/897692888725344/989703124920288
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce
- https://en.cppreference.com/w/cpp/algorithm/reduce
- https://en.cppreference.com/w/cpp/algorithm/transform