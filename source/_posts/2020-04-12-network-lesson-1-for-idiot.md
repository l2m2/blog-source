---
title: 网络白痴的基础课(一)
toc: true
date: 2020-04-12 17:43:00
description: 列举一些常见的网络术语，以及Linux下常见的与网络相关的命令
tags:
- Linux
- 网络
---

本人是网络白痴一个，写此文纯属个人备忘， 无任何指导意义。欢迎指正。

本文中所有的命令都是基于`CentOS 7` 环境测试。

## 什么是网关`Gateway`？

在维基百科中搜索`Gateway`, 你将看到下面的列表（Computing and telecommunication devices部分）。中文部分为注解。

- [Default gateway](https://en.wikipedia.org/wiki/Default_gateway), the node in a computer network that passes traffic from a local network to other networks or the Internet (router)

  **此处说的网关是一个网络层的概念。**

  PC本身不具备路由寻址能力，所以PC要把所有的IP包发送到一个默认的中转地址上面进行转发，也就是默认网关。

  此处的网关就是一个网络连接到另一个网络的关口。它可以在路由器上，也可以在交换机上，也可以在某个服务器上。

  它是某个网络访问其他网络的门，你要去其他网络，从这个门过。

- [Gateway (computer program)](https://en.wikipedia.org/wiki/Gateway_(computer_program)), a link between two computer programs allowing them to share information and bypass certain protocols on a host computer

  **此处说的网关是一个计算机程序**，用来链接两个程序，使它们可以共享信息。

- [Gateway (telecommunications)](https://en.wikipedia.org/wiki/Gateway_(telecommunications)), a network node equipped for interfacing with another network that uses different communication protocols

  网关是电信中用于电信网络的一种联网硬件，它允许数据从一个离散网络流向另一个网络。网关与路由器或交换机的区别在于，它们使用多个协议进行通信以连接一堆网络[1] [2]，并且可以在开放系统互连模型（OSI）的七个层中的任何一层上运行。

  **此处说的网关是一个硬件。**

## 什么是子网掩码`subnet mask`？

就是用掩码的方式来表示子网。

通常我们看到这样的子网掩码`255.255.255.0`, 它是啥意思呢？

它表示一个子网，即一堆主机组成的网络。



## 路由

## 查看/修改IP

通过 `ifconfig`查看

```bash
$ ifconfig
eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.21.2.192  netmask 255.255.255.0  broadcast 10.21.2.255
        inet6 fe80::d0f3:8aef:5db4:842a  prefixlen 64  scopeid 0x20<link>
        ether 08:94:ef:9a:b5:b6  txqueuelen 1000  (Ethernet)
        RX packets 3159393  bytes 1443056522 (1.3 GiB)
        RX errors 0  dropped 1343  overruns 0  frame 0
        TX packets 2467562  bytes 943094455 (899.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eno2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.28.88.12  netmask 255.255.255.0  broadcast 10.28.88.255
        inet6 fe80::a105:98f3:8d0:2513  prefixlen 64  scopeid 0x20<link>
        ether 08:94:ef:9a:b5:b7  txqueuelen 1000  (Ethernet)
        RX packets 9229041  bytes 1837599767 (1.7 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1476740  bytes 779957204 (743.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

通过 `ip a`查看

```bash
$ ip a
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 08:94:ef:9a:b5:b6 brd ff:ff:ff:ff:ff:ff
    inet 10.21.2.192/24 brd 10.21.2.255 scope global noprefixroute eno1
       valid_lft forever preferred_lft forever
    inet6 fe80::d0f3:8aef:5db4:842a/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: eno2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 08:94:ef:9a:b5:b7 brd ff:ff:ff:ff:ff:ff
    inet 10.28.88.12/24 brd 10.28.88.255 scope global noprefixroute eno2
       valid_lft forever preferred_lft forever
    inet6 fe80::a105:98f3:8d0:2513/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

修改IP

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-eno1
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eno1
UUID=e6977585-7a95-4ff1-9598-b06af7ee8bc5
DEVICE=eno1
ONBOOT=yes
IPADDR=10.21.2.192
PREFIX=24
GATEWAY=10.21.2.254
DNS1=10.3.1.6
DNS2=10.3.1.2
ZONE=public
$ systemctl restart network
```

你也可以通过`ifup`和`ifdown`命令来激活和禁用指定的网络接口。

## 查看/修改路由

通过`route`命令来查看路由表

```bash
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.21.2.254     0.0.0.0         UG    100    0        0 eno1
10.21.2.0       0.0.0.0         255.255.255.0   U     100    0        0 eno1
10.28.0.0       10.28.88.254    255.255.0.0     UG    0      0        0 eno2
10.28.88.0      0.0.0.0         255.255.255.0   U     101    0        0 eno2
```

可以看到，默认网关是10.21.2.254。到10.28.0.0网络的网关是10.28.88.254。

通过`route add`来添加路由。

```bash
$ route add -net 10.28.0.0/16 gw 10.28.88.254
```

也可以通过`route del`来删除路由。

## Reference

- https://www.cyberciti.biz/faq/howto-setting-rhel7-centos-7-static-ip-configuration/
- https://www.zhihu.com/question/21787311
- https://en.wikipedia.org/wiki/Gateway
- https://www.zhihu.com/question/56895036
- https://mp.weixin.qq.com/s/jAITB4o1nnO5M2wt0hDqjw