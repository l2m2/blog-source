---
title: 给Vue项目添加全屏支持
toc: false
date: 2021-02-02 09:11:00
description: 使用screenfull.js
tags:
- Vue
- Vuetify
---

## 安装

```bash
$ yarn add screenfull
```

## 组件

`components/Screenfull.vue`

```vue
<template>
  <v-btn icon @click="onFullscreenClicked">
    <v-icon>
      {{ isFullscreen ? "mdi-fullscreen-exit" : "mdi-fullscreen" }}
    </v-icon>
  </v-btn>
</template>
<script>
import screenfull from "screenfull";
export default {
  name: "Screenfull",
  data: () => ({
    isFullscreen: false
  }),
  mounted() {
    if (screenfull.isEnabled) {
      screenfull.on("change", this.changeFullscreen);
    }
  },
  methods: {
    onFullscreenClicked() {
      if (screenfull.isEnabled) {
        screenfull.toggle();
      }
    },
    changeFullscreen() {
      this.isFullscreen = screenfull.isFullscreen;
    }
  },
  destroyed() {
    screenfull.off("change", this.changeFullscreen);
  }
};
</script>
```

在需要使用的地方添加此组件即可。

```vue
<template>
  <v-app id="inspire">
    <v-app-bar app>
      <v-app-bar-nav-icon @click="toggleNavi()"></v-app-bar-nav-icon>
      <v-toolbar-title>{{ $t("app.name") }}</v-toolbar-title>
      <v-spacer />
      <Screenfull />
    </v-app-bar>
    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>
<script>
export default {
  components: {
    Screenfull: () => import("./components/Screenfull")
  }
}
</script>
```

## Reference

- https://github.com/sindresorhus/screenfull.js
- https://github.com/flipped-aurora/gin-vue-admin

