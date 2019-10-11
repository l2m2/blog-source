---
title: JavaScript中的柯里化和偏函数
toc: false
date: 2019-07-08 16:56:00
description: Currying & Partial
tags:
- JavaScript
---
## 柯里化(Currying)

柯里化是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

```js
//Un-curried function
function unCurried(x, y) {
  return x + y;
}

//Curried function
function curried(x) {
  return function(y) {
    return x + y;
  }
}
curried(1)(2) // Returns 3
```

## 偏函数(Partial)

偏函数是把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。

```js
//Impartial function
function impartial(x, y, z) {
  return x + y + z;
}
var partialFn = impartial.bind(this, 1, 2);
partialFn(10); // Returns 13
```

## Reference

- [freecodecamp](https://www.freecodecamp.org/)

