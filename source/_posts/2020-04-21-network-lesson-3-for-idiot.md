---
title: 我是网络白痴(三)：添加永久路由
toc: false
date: 2020-04-21 15:43:00
description: 修改/etc/sysconfig/network-scripts/route-${interface}
tags:
- CentOS
- 网络
---

本人是网络白痴一个，写此文纯属个人备忘， 无任何指导意义。欢迎指正。
本文中所有的命令都是基于`CentOS 7` 环境测试。



[上次](https://l2m2.top/2020/04/12/2020-04-12-network-lesson-1-for-idiot/) 用`route add`添加了路由，结果重启服务器后，路由没了。

然后google到了另一个命令：`ip`。

`ip`命令整合了`ifconfig`和`route`的功能，更牛批。

用`ip route`查看路由。

```bash
$ ip route
default via 10.21.2.254 dev eno1 proto static metric 100 
10.21.2.0/24 dev eno1 proto kernel scope link src 10.21.2.192 metric 100 
10.28.0.0/16 via 10.28.88.254 dev eno2 
10.28.88.0/24 dev eno2 proto kernel scope link src 10.28.88.12 metric 101 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 
```

用`ip route add`添加静态路由。

```bash
$ ip route add 10.28.0.0/16 via 10.28.88.254
```

那么如何永久生效呢？

可以通过编辑/etc/sysconfig/network-scripts/route-eno2文件，为eno2添加其他静态路由，如下所示

```bash
$ cat /etc/sysconfig/network-scripts/route-eno2 
10.28.0.0/16 via 10.28.88.254
```

## Reference

- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/deployment_guide/s1-networkscripts-static-routes
- https://linoxide.com/how-tos/howto-permanently-add-static-route-in-linux/
- http://linux.vbird.org/linux_server/0140networkcommand.php#ip_cmd
- https://www.cyberciti.biz/faq/linux-route-add/

