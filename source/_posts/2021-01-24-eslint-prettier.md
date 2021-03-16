---
title: 使用ESLint+Prettier来工程化你的Vue代码
toc: false
date: 2021-01-24 16:36:00
description: ...
tags:
- VS Code
- Vue
---

## ESLint

ESLint是**静态代码检查工具**。

一般地，我们在使用Vue脚手架创建Vue前端工程时都会启动ESLint静态代码检查。

使用`yarn lint` or `npm run lint`会默认自动修复ESLint错误。

那怎样做到在保存时就自动修复呢？

1. 安装 VS Code 插件 ESLint

2. 在当前工程的根目录下的.vscode/settings.json添加ESLint插件的配置

   ```json
   {
       "editor.codeActionsOnSave": {
           "source.fixAll.eslint": true
       },
       "eslint.workingDirectories": [
           { "mode": "auto" }
       ]
   }
   ```

## Prettier

Prettier是**代码格式化工具。**

## 最佳实践：ESLint+Prettier

1. 将.vscode从你的.gitignore中去除，以保存每个成员都使用同样的配置

2. 在.vscode/extensions.json中增加推荐安装的插件列表

   ```json
   {
       "recommendations": [
         	"dbaeumer.vscode-eslint"
       ]
   }
   ```

   这样，新加入团队的成员可以得到推荐插件的提示，以方便的添加需要的插件。

   ![](/images/eslint-autofix-on-save-1.png)

3. **使用ESLint + Prettier的组合**

   若使用脚手架创建项目时没有选择Prettier，可以单独添加。

   ```bash
   $ yarn add --dev prettier @vue/eslint-config-prettier eslint-plugin-prettier
   ```

   修改.eslintrc.js

   ```js
   extends: ["plugin:vue/essential", "eslint:recommended", "@vue/prettier"]
   ```

3. 一个定制ESLint和Prettier的规则示例

   `.eslintrc.js`

   ```js
   module.exports = {
     root: true,
     env: {
       node: true
     },
     extends: ["plugin:vue/essential", "eslint:recommended", "@vue/prettier"],
     parserOptions: {
       parser: "babel-eslint"
     },
     rules: {
       "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
       "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off",
       "max-len": ["error", { "code": 140 }],
       "prettier/prettier": [ "error", { printWidth: 140} ]
     }
   };
   ```


## Reference

- https://eslint.org/docs/rules/

- https://github.com/prettier/eslint-plugin-prettier