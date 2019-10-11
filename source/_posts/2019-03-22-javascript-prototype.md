---
title: 理解JavaScript中的原型和原型链
toc: true
date: 2019-03-22 13:14:54
description: 理解JavaScript原型是前端开发人员的必修课。
categories: 我爱前端
tags: 
- web
- JavaScript
---
## 基础概念

- 对象
- 函数对象
- 原型对象

下图摘取自[彻底理解JavaScript原型](https://www.cnblogs.com/wilber2013/p/4924309.html)

![](/images/js-prototype-4.png)

 

## 原型

- 每个对象都有`__proto__`属性，对应对象的原型

	> `__proto__`属性是非标准的访问器。只有部分浏览器提供了这个属性。
	
	![](/images/js-prototype-2.png) 
	
- 函数对象都有`prototype`属性，该属性的值会被赋值给该函数创建的对象的`__proto__`属性

  ![](/images/js-prototype-5.png) 

- 原型对象有`constructor`属性，这个属性对应创建所有指向该原型的实例的构造函数

  ![](/images/js-prototype-3.png)  

- `{}`创建的对象的原型是`Object`

  ![](/images/js-prototype-6.png) 

## 原型链

因为每个对象和原型都有原型，对象的原型指向对象的父，而父的原型又指向父的父，这种原型层层连接起来的就构成了原型链。

![](/images/js-prototype-7.png) 

## 用原型实现继承

```js
function Foo() { 
    this.name = "Foo"; 
}
Foo.prototype.getName = function () { 
    console.log("I am " + this.name); 
}

function Bar() {}
// Set the Bar's prototype to Foo's constructor, thus inheriting all of Foo.prototype methods and properties.
Bar.prototype = new Foo();
let bar = new Bar(); 
bar.getName(); // => I am Foo
```

Or

```js
function Animal() { }
Animal.prototype = {
  constructor: Animal,
  eat: function() {
    console.log("nom nom nom");
  }
};

function Dog() { }
Dog.prototype = Object.create(Animal.prototype);

let beagle = new Dog();
beagle.eat();  // => "nom nom nom"
```

请注意差别：

```js
function Foo() { 
    this.name = "Foo"; 
}
Foo.prototype.getName = function () { 
    console.log("I am " + this.name); 
}

function Bar() {}
Bar.prototype = new Foo();
let bar = new Bar(); 
bar.getName(); // => I am Foo

function Bar2() {}
Bar2.prototype = Object.create(Foo.prototype);
let bar2 = new Bar2(); 
bar2.getName(); // => I am undefined
```

这是由`new `和`Object.create()`的差别引起。

```
Very simply said, new X is Object.create(X.prototype) with additionally running the constructor function. (And giving the constructor the chance to return the actual object that should be the result of the expression instead of this.)
```

摘自[Understanding the difference between Object.create() and new SomeFunction()](https://stackoverflow.com/questions/4166616/understanding-the-difference-between-object-create-and-new-somefunction)

## 常见问题

- `__proto__`和`prototype`是同一个东西吗？

  **不是！**

  每个对象都有`__proto__`属性，它对应该对象的原型。

  只有函数对象才有`prototype`属性，当一个函数被用作构造函数来创建实例时，该函数的prototype属性值将被作为原型赋值给所有对象实例（也就是设置实例的`__proto__`属性）。

  但是 `Function.prototype === Function.__proto__` 为 `true`。

## Reference

- [JavaScript Prototype in Plain Language](http://javascriptissexy.com/javascript-prototype-in-plain-detailed-language/)
- [彻底理解JavaScript原型](https://www.cnblogs.com/wilber2013/p/4924309.html)



