---
title: 在Vue项目中使用iconfont
toc: false
date: 2021-03-31 13:32:00
description: ...
tags:
- Vue.js
- iconfont
---

## iconfont

是阿里推出的很强大且图标内容很丰富的矢量图标库，提供矢量图标下载、在线存储、格式转换等功能。

## 在Vue项目使用

1. 登录 https://www.iconfont.cn/

2. 进入资源管理->我的项目

3. 新建项目
   ![](/images/use-iconfont-1.png)

4. 搜索图标，将选中的图标添加入库

   ![](/images/use-iconfont-2.png)

5. 将图标添加至项目

   ![](/images/use-iconfont-3.png)

6. 生成代码

   ![](/images/use-iconfont-4.png)

7. 全局注册IconFont组件

   `plugins/iconfont.js`

   ```js
   import Vue from "vue";
   import { Icon } from "ant-design-vue";
   
   const IconFont = Icon.createFromIconfontCN({
     scriptUrl: "//at.alicdn.com/t/font_2456846_xduwpbra1v.js"
   });
   
   Vue.component("icon-font", IconFont);
   ```

   上面的scriptUrl可在第6步中生成的代码中获得


8. 在main.js中引入

   `main.js`

   ```js
   import "./plugins/iconfont";
   ```
   

9. 若添加或更新了图标，需再次生成代码，替换scriptUrl即可。

## 其他

1. 若需要离线使用，下载到本地即可
2. 也可以以css的方式引入

## Reference

- https://www.iconfont.cn/
- https://antdv.com/components/icon-cn/#components-icon-demo-use-iconfont.cn