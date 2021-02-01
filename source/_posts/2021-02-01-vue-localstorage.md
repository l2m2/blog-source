---
title: 在Vue项目中使用LocalStorage
toc: false
date: 2021-02-01 22:25:00
description: 使用vuex-persist包方便的持久化，而无需显式操作LocalStorage
tags:
- Vue
---

使用vex-persist对vuex持久化到LocalStorage, 这样我们就无需显式的对LocalStorage进行操作了。

## 安装

```bash
$ yarn add vuex-persist
```

## Store

`store/index.js`

```js
import VuexPersistence from "vuex-persist";
const vuexLocal = new VuexPersistence({
  storage: window.localStorage,
  modules: ["global", "user"]
});
export default new Vuex.Store({
  state: {},
  mutations: {},
  actions: {},
  modules: {
    global,
    user
  },
  plugins: [vuexLocal.plugin]
});
```

这样，就把global和user两个模块的信息自动存储到LocalStorage了。

![](/images/vue-localstorage-1.png)

存储的key是vex, value长这样：

```json
{
  "global": {
    "lang": "en"
  },
  "user": {
    "token": "eyJhbGci"
  }
}
```

## Reference

- https://cn.vuejs.org/v2/cookbook/client-side-storage.html
- https://robinck.gitbooks.io/vue-storage/content/
- https://championswimmer.in/vuex-persist/