---
title: 解决“Gradle sync failed: unable to load class xxx”的错误
toc: false
date: 2021-06-09 16:27:00
description: ...
tags:
- Android
---

使用Gradle构建工程的时候，出现了如下错误：

```
Gradle sync failed: unable to load class 'sync_studio_tooling_boskacbmhla13vwj7dekf1yha'.
```

**可能的原因以及对应的解决方案**

1. Gradle和Gradle plugin版本没有对应。

   这是最常见的情况。

   解决方案：修改为正确的对应版本。

   修改工程目录下的*build.gradle*中Gradle插件的版本, 例如修改为`4.0.1`。

   ```
   // Top-level build file where you can add configuration options common to all sub-projects/modules.
   buildscript {
       repositories {
           google()
           mavenCentral()
       }
       dependencies {
           classpath "com.android.tools.build:gradle:4.0.1"
   
           // NOTE: Do not place your application dependencies here; they belong
           // in the individual module build.gradle files
       }
   }
   ```

   再修改工程目录下的*gradle/wrapper/gradle-wrapper.properties*中Gradle的版本，例如修改为`6.1.1`。

   ```
   distributionBase=GRADLE_USER_HOME
   distributionUrl=https\://services.gradle.org/distributions/gradle-6.1.1-bin.zip
   distributionPath=wrapper/dists
   zipStorePath=wrapper/dists
   zipStoreBase=GRADLE_USER_HOME
   ```

   >  Gradle是Android用来build的工具， 而Gradle plugin是封装好的可以复用的Gradle script。

2. Gradle的缓存错误

   因为网络或其他原因造成Gradle的缓存错误。
   
   解决方案：删除缓存，重新来过。
   
   ```shell
   $ rm -rf ~/.gradle/
   ```
   
   

## Reference

- https://www.tehrir.com/gradle-sync-failed

