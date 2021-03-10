---
title: 解决Vue项目中无法跳转到@/path的问题
toc: false
date: 2021-03-10 16:17:00
description: ...
tags:
- Vue
- VS Code
---

在使用VS Code开发Vue项目时，可能会遇到无法跳转到 `@/path` 的问题。以下是解决方法。

在前端项目的目录下增加`jsconfig.js`

```js
{
  "compilerOptions": {
    "target": "es6",
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "src/*"
      ]
    }
  },
  "exclude": [
    "node_modules",
    "dist"
  ],
  "include": [
    "src/**/*"
  ]
}
```

具体的用法在[这里](https://code.visualstudio.com/docs/languages/jsconfig)

如果你的工程中前端目录并不在根目录，则需要在`vetur.config.js`中指定项目，就像下面这样。

```js
module.exports = {
  projects: [
    "./frontend"
  ]
}
```

作为参考，我的工程目录长这样：

```
|--backend/
|--frontend/
|  |--src/
|  |--jsconfig.js
|--vetur.config.js
```

## Reference

- https://vuejs.github.io/vetur/reference/#example
- https://code.visualstudio.com/docs/languages/jsconfig