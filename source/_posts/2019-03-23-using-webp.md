---
title: 使用WebP优化图片加载效率
description: WebP是一种图像格式，可为Web上的图像提供卓越的无损和有损压缩。
toc: true
date: 2019-10-11 11:33:29
categories: 我爱前端
tags: webp
---
## 1 什么是WebP?

WebP is a modern **image format** that provides superior **lossless and lossy** compression for images on the web. Using WebP, webmasters and web developers can create smaller, richer images that make the web faster.

WebP是一种图像格式，可为Web上的图像提供卓越的无损和有损压缩。使用WebP，Web开发人员可以创建更小，更丰富的图像，以提升网站的图片加载效率。

## 2 Can I Use?

详见[Can I Use WebP](<https://caniuse.com/#search=webp>).

![](/images/webp-1.png)

### 2.1 有损

- Google Chrome (desktop) 17+
- Google Chrome for Android version 25+
- Microsoft Edge 18+
- Firefox 65+
- Opera 11.10+
- Native web browser, Android 4.0+ (ICS)

### 2.2 有损、无损、Alpha

- Google Chrome (desktop) 23+
- Google Chrome for Android version 25+
- Microsoft Edge 18+
- Firefox 65+
- Opera 12.10+
- Native web browser, Android 4.2+ (JB-MR1)
- Pale Moon 26+

### 2.3 Animation

- Google Chrome (desktop and Android) 32+
- Microsoft Edge 18+
- Firefox 65+
- Opera 19+

## 3 怎么玩？

### 3.1 安装WebP工具

到[https://developers.google.com/speed/webp/docs/precompiled](https://developers.google.com/speed/webp/docs/precompiled)根据平台选择对应的包下载即可。

### 3.2 转换为WebP格式的图片

```
cwebp bg.png -o bg.webp
```

大小对比：

bg.png: 238 KB 

bg.webp: 101 KB

### 3.3 转换目录下所有的图片

写个脚本就好啦。

下面是windows下的批处理示例。

```
:: batch usage: https://ss64.com/nt/
:: Command Line arguments (Parameters): https://ss64.com/nt/syntax-args.html
@echo off
cd /d %~dp0
:: /r Loop through files (Recurse subfolders)
:: %~dp1 Expand %1 to a drive letter and path only
:: %~n1 Expand %1 to a file Name without file extension or path - MyFile 
:: or if only a path is present, with no trailing backslash, the last folder in that path.
for /r %%i in (*.jpg, *.png) do ( cwebp %%i -o %%~dpi%%~ni.webp )
```

### 3.5 使用webp

本文只介绍在前端处理的方法。

#### 3.5.1 方法一：监测是否支持，然后替换attr

给所有用到图片的标签加上webp的class, 并将图片地址存在data-*中。

```html
<img class="webp" data-src="assets/images/qq.jpg" width="100" height="100"  alt="">
<a class="work-item default-popup webp" data-href="assets/images/work/big-1.jpg">
<div class="bg-cms webp" data-bg="assets/images/bg.png"></div>
```

DOM加载完成再用JS替换。

```js
;(function(doc){
  // check_webp_feature:
  //   'feature' can be one of 'lossy', 'lossless', 'alpha' or 'animation'.
  //   'callback(feature, result)' will be passed back the detection result (in an asynchronous way!)
  function check_webp_feature(feature, callback) {
    var kTestImages = {
        lossy: "UklGRiIAAABXRUJQVlA4IBYAAAAwAQCdASoBAAEADsD+JaQAA3AAAAAA",
        lossless: "UklGRhoAAABXRUJQVlA4TA0AAAAvAAAAEAcQERGIiP4HAA==",
        alpha: "UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==",
        animation: "UklGRlIAAABXRUJQVlA4WAoAAAASAAAAAAAAAAAAQU5JTQYAAAD/////AABBTk1GJgAAAAAAAAAAAAAAAAAAAGQAAABWUDhMDQAAAC8AAAAQBxAREYiI/gcA"
    };
    var img = new Image();
    img.onload = function () {
        var result = (img.width > 0) && (img.height > 0);
        callback(feature, result);
    };
    img.onerror = function () {
        callback(feature, false);
    };
    img.src = "data:image/webp;base64," + kTestImages[feature];
  }

  check_webp_feature('lossy', function(feature, support){
    $('.webp').each(function(){
      if ($(this).is('img')) {
        var src = $(this).attr('data-src');
        if (support){
          src = src.replace(/\.jpg|\.png$/, '.webp');
        }
        $(this).attr('src', src);
      } else if ($(this).is('a')){
        var href = $(this).attr('data-href');
        if (support){
          href = href.replace(/\.jpg|\.png$/, '.webp');
        }
        $(this).attr('href', href);
      } else if ($(this).is('div')) {
        var data_bg =  $(this).attr('data-bg');
        if (support){
          data_bg = data_bg.replace(/\.jpg|\.png$/, '.webp');
        }
        $(this).attr('data-bg', data_bg);
      }
    });
  });
}(document));
```

详细源码见[github](https://github.com/l2m2/dirty-projects/tree/master/personal-page-demo)。

## 4 Reference

- [A new image format for the Web](https://developers.google.com/speed/webp/)

- [webp 图片牛刀小试](https://juejin.im/entry/583a9cab61ff4b007ecbf597)

- [把网站的图片升级到WebP格式吧](https://segmentfault.com/a/1190000007482148)

  