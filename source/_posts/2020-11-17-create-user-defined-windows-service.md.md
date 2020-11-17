---
title: 手动创建Windows服务
toc: false
date: 2020-11-17 11:07:00
description: 通过instsrv.exe和srvany.exe手动创建一个Windows服务。
tags:
- Windows
---

本文介绍如何通过`instsrv.exe`和`srvany.exe`手动创建一个Windows服务。

`instsrv.exe`： 服务注册工具。

`srvany.exe`: 注册程序的服务外壳。

你可以在[这里](https://github.com/l2m2/resource/tree/master/instsrv)下载它们。

**操作步骤**

1. 以管理员身份打开命令行窗口。

2. 创建服务。

   ```bash
   $ c:\srvinst\instsrv.exe toppdtserver c:\srvinst\srvany.exe
   
   The service was successfuly added!
   
   Make sure that you go into the Control Panel and use
   the Services applet to change the Account Name and
   Password that this newly installed service will use
   for its Security Context.
   
   ```

   其中toppdtserver是服务名称。

3. 打开注册表并找到子项 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\toppdtserver`

4. 右键`toppdtserver`节点，新建项`Parameters`

   ![](/images/create-user-defined-windows-service-2.png)

5. 选中`Parameters`节点，在右侧新建字符串值。

   名称：Application, 类型：REG_SZ, 值为应用程序的路径。

   ![](/images/create-user-defined-windows-service-3.png)

6. 关闭注册表编辑器。

7. 在服务列表中启动服务。

   ![](/images/create-user-defined-windows-service-1.png)

8. 大功告成！

## Reference

- https://docs.microsoft.com/en-us/troubleshoot/windows-client/deployment/create-user-defined-service