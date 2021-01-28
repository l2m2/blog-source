---
title: 给Vue项目添加全局的消息条
toc: false
date: 2021-01-28 22:38:00
description: Vue / Vuetify / Snackbar
tags:
- Vue
- Vuetify
---

本文演示一下如何为Vue/Vuetify项目添加全局的消息条（也叫toasts, snackbar）.

我们期待的用法大概长这样

```js
this.$q.showMessage(...)
```

## Store

把数据存在store中

`store/snackbar.js`

```js
const state = () => ({
  content: "",
  color: ""
});
const mutations = {
  showMessage(state, payload) {
    state.content = payload.content;
    state.color = payload.color;
  }
};
export default {
  namespaced: true,
  state,
  mutations
};
```

并在`store/index.js`中引入它

```js
export default new Vuex.Store({
  state: {},
  mutations: {},
  actions: {},
  modules: {
    snackbar
  }
});
```

## Plugin

为了在全局都可以使用，我们需要创建一个插件

`plugins/snackbar.js`

```js
const snackbarPlugin = {
  install: (Vue, { store }) => {
    if (!store) {
      throw new Error("Please provide vuex store.");
    }
    Vue.prototype.$q = {
      showMessage: function({ content = "", color = "" }) {
        store.commit(
          "snackbar/showMessage",
          { content, color },
          { root: true }
        );
      }
    };
  }
};
export default snackbarPlugin;
```

需要在注册到Vue的全局对象上

`main.js`

```js
import snackbarPlugin from "./plugins/snackbar";
Vue.use(snackbarPlugin, { store });
```

## Component

消息条的组件则直接用Vuetify的就好了。

`components/Snackbar.vue`

```vue
<template>
  <v-snackbar v-model="show" :color="color">
    {{ message }}
    <v-btn text @click="show = false">Close</v-btn>
  </v-snackbar>
</template>
<script>
export default {
  data() {
    return {
      show: false,
      message: "",
      color: ""
    };
  },

  created() {
    this.$store.subscribe((mutation, state) => {
      if (mutation.type === "snackbar/showMessage") {
        this.message = state.snackbar.content;
        this.color = state.snackbar.color;
        this.show = true;
      }
    });
  }
};
</script>
```

## Layout

把组件添加到应用中

`App.vue`

```vue
<template>
  <div>
    <router-view />
    <Snackbar></Snackbar>
  </div>
</template>
<script>
export default {
  name: "App",
  components: {
    Snackbar: () => import("@/components/Snackbar.vue")
  }
};
</script>
```

## Usage

可以在任意页面中使用啦。

```js
this.$q.showMessage({content: "xxx", color: "info"})
```

在axios的response钩子中提示错误：

`utils/request.js`

```js
import axios from "axios";
import context from "@/main";
const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API,
  timeout: 6000
});
service.interceptors.response.use(
  response => {
    const res = response.data;
    return res;
  },
  async error => {
    context.$q.showMessage({
      content: error.response.data.detail,
      color: "error"
    });
    return Promise.reject(error);
  }
);
export default service;
```

## Reference

- https://dev.to/stephannv/how-to-create-a-global-snackbar-using-nuxt-vuetify-and-vuex-1bda

