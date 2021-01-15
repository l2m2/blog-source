---
title: 手把手创建Vuetify项目
toc: false
date: 2021-01-15 16:21:00
description: ...
tags:
- Vue
---

**测试版本**

Vue: 2.6.x

Vuetify: 2.2.x

**用脚手架创建Vue项目**

```bash
$ vue --version 
@vue/cli 4.5.10
$ vue create vuetify-demo
Vue CLI v4.5.10
? Please pick a preset: 
  Default ([Vue 2] babel, eslint) 
  Default (Vue 3 Preview) ([Vue 3] babel, eslint) 
❯ Manually select features 
```

手动选择功能

```bash
? Check the features needed for your project: 
 ◉ Choose Vue version
 ◉ Babel
 ◯ TypeScript
 ◉ Progressive Web App (PWA) Support
 ◉ Router
 ◉ Vuex
 ◉ CSS Pre-processors
 ◉ Linter / Formatter
❯◉ Unit Testing
 ◯ E2E Testing
```

选择需要的功能后回车

```bash
? Choose a version of Vue.js that you want to start the project with (Use arrow keys)
❯ 2.x 
  3.x (Preview) 
? Use history mode for router? (Requires proper server setup for index fallback in production) (Y/n) y
? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): 
  Sass/SCSS (with dart-sass) 
❯ Sass/SCSS (with node-sass) 
  Less 
  Stylus 
? Pick a linter / formatter config: 
  ESLint with error prevention only 
  ESLint + Airbnb config 
❯ ESLint + Standard config 
  ESLint + Prettier 
? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯◉ Lint on save
 ◯ Lint and fix on commit
? Pick a unit testing solution: 
  Mocha + Chai 
❯ Jest 
? Where do you prefer placing config for Babel, ESLint, etc.? (Use arrow keys)
❯ In dedicated config files 
  In package.json 
```

进入创建的目录下，添加Vuetify

```bash
$ cd vuetify-demo/
$ vue add vuetify
? Choose a preset: (Use arrow keys)
❯ Default (recommended) 
  Prototype (rapid development) 
  Configure (advanced) 
```

跑起来

```bash
$ yarn serve
```

