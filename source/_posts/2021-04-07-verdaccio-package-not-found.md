---
title: 解决Verdaccio找不到包的问题
toc: false
date: 2021-04-07 08:56:00
description: ...
tags:
- Node.js
- Verdaccio
---

同事告诉我说，npm无法下载一个私有包，返回错误码500。

1. 试了一下，确实如此。

   ```shell
   $ npm install -g nrm # 用nrm来切换镜像
   $ nrm add toplinker http://140.197.105.14:4873/ # 添加私有镜像
   $ nrm use toplinker
   $ npm install @topibd/core
   # 下载失败
   ```

2. 怀疑是不是需要登录。

   ```shell
   $ npm login
   $ npm install @topibd/core
   # 仍然失败
   ```

3. 检查一下verdaccio的配置，我们的verdaccio是在docker中的

   ```shell
   $ ssh xxx@xxx
   $ docker exec -it verdaccio sh
   $ cat /verdaccio/conf/config.yaml
   packages:
     '@*/*':
       # scoped packages
       access: $all
       publish: $authenticated
       proxy: npmjs
   ```

   并没有做限制。


4. 通过网页打开 http://140.197.105.14:4873

   首页能显示所有包，但是打开包的详情页会出现“Oops, The package you are trying to access does not exist.”

5. 查看了下docker内的包文件，都在

   ```shell
   $ docker exec -it verdaccio sh
   $ cd /verdaccio/storage/
   ```

6. Google了下，说可能是文件夹权限的问题。试一下

   ```shell
   $ docker exec --user root -it verdaccio sh
   $ chown verdaccio: /verdaccio/ -R
   ```

7. 再试一下，成功了。

## Reference

- https://github.com/verdaccio/verdaccio/issues/483

