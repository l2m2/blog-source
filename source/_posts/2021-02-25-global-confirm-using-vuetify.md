---
title: 给Vue项目添加全局的确认对话框
toc: false
date: 2021-02-25 16:38:00
description: Vue / Vuetify / Confirm
tags:
- Vue
- Vuetify
---

本文演示一下如何为Vue/Vuetify项目添加全局的确认对话框。

我们期待的用法大概长这样

```js
this.$confirm(title, message, options)
this.$confirmDelete()
this.$confirmSave()
```

## Component

使用v-dialog定制一个确认对话框。

`components/Confirm.vue`

```vue
<template>
  <v-dialog
    v-model="dialog"
    :max-width="options.width"
    :style="{ zIndex: options.zIndex }"
    @keydown.esc="cancel"
  >
    <v-card>
      <v-card-title class="headline">
        {{ title }}
      </v-card-title>
      <v-card-text v-show="!!message"> {{ message }} </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn :color="options.yesColor" text @click.native="agree">
          {{ options.yesText }}
        </v-btn>
        <v-btn :color="options.noColor" text @click.native="cancel">{{
          options.noText
        }}</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
<script>
export default {
  data: () => ({
    dialog: false,
    resolve: null,
    reject: null,
    message: null,
    title: null,
    options: {
      yesText: "Yes",
      noText: "Cancel",
      yesColor: "primary darken-1",
      noColor: "grey",
      width: 500,
      zIndex: 200
    }
  }),
  methods: {
    open(title, message, options) {
      this.dialog = true;
      this.title = title;
      this.message = message;
      this.options = Object.assign(this.options, options);
      return new Promise((resolve, reject) => {
        this.resolve = resolve;
        this.reject = reject;
      });
    },
    deleteConfirm() {
      this.dialog = true;
      this.title = this.$t("_.dialog.delete.title");
      this.message = this.$t("_.dialog.delete.message");
      this.options = Object.assign(this.options, {
        yesText: this.$t("_.dialog.delete.yes"),
        noText: this.$t("_.dialog.delete.no"),
        yesColor: "red"
      });
      return new Promise((resolve, reject) => {
        this.resolve = resolve;
        this.reject = reject;
      });
    },
    agree() {
      this.resolve(true);
      this.dialog = false;
    },
    cancel() {
      this.resolve(false);
      this.dialog = false;
    }
  }
};
</script>
```

## Plugin

`plugins/confirm.js`

```js
const confirmPlugin = {
  install: (Vue, { normalConfirm, deleteConfirm }) => {
    Vue.prototype.$confirm = normalConfirm;
    Vue.prototype.$confirmDelete = deleteConfirm;
  }
};
export default confirmPlugin;
```

## Layout

将组件添加到应用中

`App.vue`

```vue
<template>
  <v-app>
    <router-view />
    <Confirm ref="confirm"></Confirm>
  </v-app>
</template>
<script>
import Vue from "vue";
import Confirm from "@/components/Confirm.vue";
import confirmPlugin from "@/plugins/confirm";
export default {
  name: "App",
  components: {
    Confirm
  },
  mounted() {
    Vue.use(confirmPlugin, {
      normalConfirm: this.$refs.confirm.open,
      deleteConfirm: this.$refs.confirm.deleteConfirm
    });
  }
};
</script>
```

## Usage

可以在任意页面中使用啦。

```js
if (await this.$confirmDelete()) {
  // yes
} else {
  // no
}
```

## Reference

- https://gist.github.com/eolant/ba0f8a5c9135d1a146e1db575276177d

