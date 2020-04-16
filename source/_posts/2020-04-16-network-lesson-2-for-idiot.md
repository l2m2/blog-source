---
title: 我是网络白痴(二)：利用firewall-cmd进行端口转发
toc: false
date: 2020-04-16 12:59:00
description: 利用firewall-cmd进行端口转发。一些常见的firewall-cmd命令。
tags:
- CentOS
- 网络
---

本人是网络白痴一个，写此文纯属个人备忘， 无任何指导意义。欢迎指正。
本文中所有的命令都是基于`CentOS 7` 环境测试。



在最近的一个业务场景中，客户端不能直接访问邮件服务器，需要通过某服务器进行端口转发。

通过服务器(10.21.2.192/10.28.88.12双网卡)的9281端口进行转发，转发到邮件服务器(10.3.1.251)的465端口。

**防火墙放行端口**

```bash
$ firewall-cmd --zone=public --add-port=9281/tcp --permanent
```

**开启伪装IP的功能**

```bash
$ firewall-cmd --add-masquerade --permanent
```

可以通过`firewall-cmd --remove-masquerade`禁用。

**端口转发设置**

```bash
$ firewall-cmd --permanent --add-forward-port=port=9281:proto=tcp:toport=465:toaddr=10.3.1.251
$ firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.3.1.251/24" masquerade'
$ firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.28.88.12/24" masquerade'
$ firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.21.2.192/24" masquerade'
```

**设置完成后重新加载防火墙**

```bash
$ firewall-cmd --reload
```

到这里，我们已经设置完成。

```bash
$ firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eno1 eno2
  sources: 
  services: ssh dhcpv6-client samba
  ports: 9281/tcp
  protocols: 
  masquerade: yes
  forward-ports: port=9281:proto=tcp:toport=465:toaddr=10.3.1.251
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="10.3.1.251/24" masquerade
	rule family="ipv4" source address="10.28.88.12/24" masquerade
	rule family="ipv4" source address="10.21.2.192/24" masquerade
```

附上一些**常见的firewall-cmd命令**，如下：

```bash
$ firewall-cmd --panic-on  # 拒绝所有流量，远程连接会立即断开，只有本地能登陆
$ firewall-cmd --panic-off  # 取消应急模式，但需要重启firewalld后才可以远程ssh
$ firewall-cmd --query-panic  # 查看是否为应急模式

$ firewall-cmd --add-service=<service name> #添加服务
$ firewall-cmd --remove-service=<service name> #移除服务

$ firewall-cmd --add-port=<port>/<protocol> #添加端口/协议（TCP/UDP）
$ firewall-cmd --remove-port=<port>/<protocol> #移除端口/协议（TCP/UDP）
$ firewall-cmd --list-ports #查看开放的端口

$ firewall-cmd --add-protocol=<protocol> # 允许协议 (例：icmp，即允许ping)
$ firewall-cmd --remove-protocol=<protocol> # 取消协议
$ firewall-cmd --list-protocols # 查看允许的协议

$ firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.2.1" accept" # 表示允许来自192.168.2.1的所有流量

$ firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.2.208" protocol value="icmp" accept" # 允许192.168.2.208主机的icmp协议，即允许192.168.2.208主机ping

$ firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.2.208" service name="ssh" accept" # 允许192.168.2.208主机访问ssh服务

$ firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.2.1" port protocol="tcp" port="22" accept" # 允许192.168.2.1主机访问22端口

$ firewall-cmd --zone=drop --add-rich-rule="rule family="ipv4" source address="192.168.2.0/24" port protocol="tcp" port="22" accept" # 允许192.168.2.0/24网段的主机访问22端口 
$ firewall-cmd --zone=drop --add-rich-rule="rule family="ipv4" source address="192.168.2.0/24" port protocol="tcp" port="22" reject" # 禁止192.168.2.0/24网段的主机访问22端口
```

## Reference

- https://www.cnblogs.com/stulzq/p/9808504.html