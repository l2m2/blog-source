---
title: VS Code保存时自动修复ESLint错误
toc: false
date: 2021-01-24 16:36:00
description: ...
tags:
- VS Code
- Vue
---

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

**团队开发最佳实践**

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

3. 使用ESLint + Prettier的组合

   若使用脚手架创建项目时没有选择Prettier，可以单独添加。

   ```bash
   $ yarn add --dev prettier @vue/eslint-config-prettier eslint-plugin-prettier
   ```

   修改.eslintrc.js

   ```js
   extends: ["plugin:vue/essential", "eslint:recommended", "@vue/prettier"]
   ```

   

