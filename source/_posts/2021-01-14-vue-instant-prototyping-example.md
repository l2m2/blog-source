---
title: Vue快速原型开发示例
toc: false
date: 2021-01-14 18:48:00
description: ...
tags:
- Vue
---

1. 全局安装@vue/cli-service-global

   ```bash
   $ yarn global add @vue/cli-service-global
   ```

2. 编写单文件example.vue

   ```vue
   <template>
       <div id="app">
           {{ message }}
       </div>
   </template>
   <script>
   export default {
       data: function() {
           return {
               message: 'Hello Vue!'
           }
       }
   }
   </script>
   <style scoped>
       div {
           background-color: red;
       }
   </style>
   ```
   
3. 跑起来

   ```bash
   $ vue serve example.vue
   ```

## Reference

- https://cli.vuejs.org/zh/guide/prototyping.html

