---
title: 在JavaScript中使用正则
toc: true
date: 2019-07-06 16:52:18
description: 本文是完成freecodecamp练习时顺手写的，非常基础
tags:
- JavaScript
- 正则
---
## 1 正则

略略略

## 2 在JavaScript中玩正则

### 2.1 `test`

test() 方法用于检测一个字符串是否匹配某个模式，如果字符串中含有匹配的文本，则返回 true，否则返回 false。

```js
let testStr = "freeCodeCamp";
let testRegex = /Code/;
testRegex.test(testStr);
// true
```

### 2.2 用`|`匹配多个字符串

```js
let petString = "James has a pet cat.";
let petString2 = "James has a pet dog.";
let petRegex = /dog|cat|bird|fish/;
let result = petRegex.test(petString);
// true
let result2 = petRegex.test(petString2);
// true
```

### 2.3 忽略大小写`i`

```js
let myString = "freeCodeCamp";
let fccRegex = /freeCodeCamp/i;
let result = fccRegex.test(myString);
```

### 2.4 用`match`提取匹配的字符串

```js
"Hello, World!".match(/Hello/);
// Returns ["Hello"]
let ourStr = "Regular expressions";
let ourRegex = /expressions/;
ourStr.match(ourRegex);
// Returns ["expressions"]
```

### 2.5 全局匹配`g`

```js
let repeatRegex = /Repeat/g;
testStr.match(repeatRegex);
// Returns ["Repeat", "Repeat", "Repeat"]
```

### 2.6 通配符`.`

```js
let exampleStr = "Let's have fun with regular expressions!";
let unRegex = /.un/; 
let result = unRegex.test(exampleStr);
// it matches the strings "run", "sun", "fun", "pun", "nun", and "bun". 
```

### 2.7`[]`

查找方括号之间的任何字符。

```js
let bigStr = "big";
let bagStr = "bag";
let bugStr = "bug";
let bogStr = "bog";
let bgRegex = /b[aiu]g/;
bigStr.match(bgRegex); // Returns ["big"]
bagStr.match(bgRegex); // Returns ["bag"]
bugStr.match(bgRegex); // Returns ["bug"]
bogStr.match(bgRegex); // Returns null
```

使用`-`

```js
let catStr = "cat";
let batStr = "bat";
let matStr = "mat";
let bgRegex = /[a-e]at/;
catStr.match(bgRegex); // Returns ["cat"]
batStr.match(bgRegex); // Returns ["bat"]
matStr.match(bgRegex); // Returns null
```

**匹配数字和字母**

```js
let jennyStr = "Jenny8675309";
let myRegex = /[a-z0-9]/ig;
// matches all letters and numbers in jennyStr
jennyStr.match(myRegex);
```

### 2.8 使用`^`匹配非

`[^abc]`查找任何不在方括号之间的字符。

例子：匹配不是数字也不是元音的字符

```js
let quoteSample = "3 blind mice.";
let myRegex = /[^0-9aeiou]/gi; 
let result = quoteSample.match(myRegex); 
```

### 2.9 量词

用`+`匹配一次或多次

```js
let difficultSpelling = "Mississippi";
let myRegex = /s+/g; 
let result = difficultSpelling.match(myRegex);
console.log(result);
// ["ss", "ss"]
```

用`*`匹配0次或多次

```js
let soccerWord = "gooooooooal!";
let gPhrase = "gut feeling";
let oPhrase = "over the moon";
let goRegex = /go*/;
soccerWord.match(goRegex); // Returns ["goooooooo"]
gPhrase.match(goRegex); // Returns ["g"]
oPhrase.match(goRegex); // Returns null
```

用`?`匹配0次或1次

```js
let american = "color";
let british = "colour";
let rainbowRegex= /colou?r/;
rainbowRegex.test(american); // Returns true
rainbowRegex.test(british); // Returns true
```

### 2.10 惰性匹配

```js
let str = "<h1>Winter is coming</h1>";
let myRegex = /<.*?>/; 
let result = str.match(myRegex);
console.log(result);
// ["<h1>"]
```

### 2.11 匹配字符串的开始和结尾

用`^`匹配开始

```js
let rickyAndCal = "Cal and Ricky both like racing.";
let calRegex = /^Cal/;
let result = calRegex.test(rickyAndCal);
```

用`$`匹配结尾

``` js
let caboose = "The last car on a train is the caboose";
let lastRegex = /caboose$/;
let result = lastRegex.test(caboose);
```

### 2.12 匹配所有字母、数字、下划线 `\w`

```js
let longHand = /[A-Za-z0-9_]+/;
let shortHand = /\w+/;
let numbers = "42";
let varNames = "important_var";
longHand.test(numbers); // Returns true
shortHand.test(numbers); // Returns true
longHand.test(varNames); // Returns true
shortHand.test(varNames); // Returns true
```

### 2.13 匹配非字母、数字、下划线 `\W`

```js
let shortHand = /\W/;
let numbers = "42%";
let sentence = "Coding!";
numbers.match(shortHand); // Returns ["%"]
sentence.match(shortHand); // Returns ["!"]
```

### 2.14 匹配所有数字 `\d`

### 2.15 匹配非数字 `\D`

### 2.16 匹配空白`\s`

匹配任何空白字符，包括空格、制表符、换页符等等。等价于` [ \f\n\r\t\v]`。

### 2.17 匹配非空白 `\S`

匹配任何非空白字符。等价于` [^\f\n\r\t\v]`。

### 2.18 用`{}`指定匹配数

用`{3,5}`表示匹配3到5次

```js
let A4 = "aaaah";
let A2 = "aah";
let multipleA = /a{3,5}h/;
multipleA.test(A4); // Returns true
multipleA.test(A2); // Returns false
```

用`{3,}`表示至少匹配3次

```js
let A4 = "haaaah";
let A2 = "haah";
let A100 = "h" + "a".repeat(100) + "h";
let multipleA = /ha{3,}h/;
multipleA.test(A4); // Returns true
multipleA.test(A2); // Returns false
multipleA.test(A100); // Returns true
```

用`{3}`表示匹配确定的3次

```js
let A4 = "haaaah";
let A3 = "haaah";
let A100 = "h" + "a".repeat(100) + "h";
let multipleHA = /ha{3}h/;
multipleHA.test(A4); // Returns false
multipleHA.test(A3); // Returns true
multipleHA.test(A100); // Returns false
```

### 2.19 lookahead

`(?=pattern)` 正向肯定预查。

在任何匹配pattern的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。

`(?!pattern)`正向否定预查。

在任何不匹配pattern的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。

```js
"qu".match(/q(?=u)/); // Returns ["q"]
"qt".match(/q(?!u)/); // Returns ["q"]
/q(?=u)/.test("qu"); // true
/q(?=u)/.test("qt"); // false
/q(?!u)/.test("qu"); // false
/q(?!u)/.test("qt"); // true
```

`lookahead`的更实际的用途是检查一个字符串中的两个或更多个模式。

下面的例子是一个简单的密码检查器，可以查找3到6个字符和至少一个数字。

```js
let password = "abc123";
let checkPass = /(?=\w{3,6})(?=\D*\d)/;
checkPass.test(password); // Returns true
```

### 2.20 使用组捕获重用模式

要指定重复字符串的显示位置，请使用反斜杠（\），然后使用数字。此数字从1开始，随着您使用的每个其他捕获组而增加。

例如用`\1`来匹配第一组。

```js
let nonRepeatStr = "regex";
let repeatStr = "regex regex";
let repeatRegex = /(\w+)\s\1/;
repeatRegex.test(repeatStr); // Returns true
repeatRegex.test(nonRepeatStr); // Returns false
repeatStr.match(repeatRegex); // Returns ["regex regex", "regex"]
```

### 2.21 使用组捕获去替换字符串

交换单词的位置

```js
"Code Camp".replace(/(\w+)\s(\w+)/, '$2 $1');
// Returns "Camp Code"
```

去掉首尾的空格（与trim等效）

```js
let hello = "   Hello, World!  ";
let wsRegex = /^\s+|\s+$/g; 
let result = hello.replace(wsRegex, ""); 
```

## 3 Reference

- [freecodecamp](https://www.freecodecamp.org/)

