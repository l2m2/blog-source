---
title: 安装PM2为Windows服务
toc: false
date: 2020-11-16 18:09:00
description: 安装PM2为windows服务实战记录。
tags:
- Node.js
- PM2
---

## PM2

PM2是一个进程管理工具，通常我们用它来管理node进程。

详细请参考[官网文档](https://pm2.keymetrics.io/docs/usage/quick-start/)。

## pm2-windows-service

[pm2-windows-service](https://www.npmjs.com/package/pm2-windows-service) 是一个NPM包，用它可以轻松地在Windows机器上安装PM2为一个Windows服务。

**安装NPM包**

```bash
$ npm i pm2-windows-service -g
```

**安装服务**

```bash
$ pm2-service-install
```

安装服务后，在服务列表中可以查看到此服务。

![](images/pm2-windows-service-1.png)

在执行`pm2 start`之后记得保存状态。

```bash
$ pm2 save
```

**卸载服务**

```bash
$ pm2-service-uninstall
```

### 常见问题

1. 执行`pm2-service-install`后，在服务列表中可以查看到服务，但服务的状态不是正在运行。

   右键服务列表中的PM2, 查询到Windows服务启动的路径，进入该目录（例如：`C:\Users\toplinker\AppData\Roaming\npm\node_modules\pm2-windows-service\src\daemon`）。

   ![](images/pm2-windows-service-2.png)

   在该目录下，有一个日志文件`pm2.err.log`，此文件详细记录了该windows服务启动失败的原因。可以根据原因逐步排查。

   **a) 如果日志中提示 Sorry, this script requires pm2.**

   这种情况下大概率是PM2的路径没有正确配置，可手动修改`dump.pm2`文件中的`PM2_SERVICE_PM2_DIR`,并删除`PM2_SERVICE_SCRIPTS`, 例如：

   ```json
   {
       "PM2_SERVICE_PM2_DIR": "C:\\Users\\toplinker\\AppData\\Roaming\\npm\\node_modules\\pm2"
   }
   ```

   `dump.pm2`文件是在pm2-service-install执行过程中指定的，它的路径取决于你的指定。

2. 在服务列表中可以查看到服务，服务也是正在运行的状态。但重启电脑后，`pm2 list`

   中并没有之前启动的进程。
   
   请确认有正确执行 `pm2 save`.
   
   ```bash
   $ pm2 start server.js
   $ pm2 save
   ```
   
3.  如果之前已经安装了类似包，建议先卸载之前安装的npm包。若卸载后服务列表中仍然存在，可手动删除服务。

   ```bash
   $ sc delete pm2.exe # pm2.exe是服务的名称，可在服务列表右键中查看
   ```

## Reference

- https://pm2.keymetrics.io/
- https://www.npmjs.com/package/pm2-windows-service
- https://github.com/jon-hall/pm2-windows-service/issues/43