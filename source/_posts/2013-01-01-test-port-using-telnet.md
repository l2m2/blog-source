---
title: Windows使用telnet测试端口
toc: false
date: 2013-01-01 16:46:58
description: 端口测试
tags:
- Windows
---
## 简介

Telnet是一个应用层协议。使用于[互联网](https://zh.wikipedia.org/wiki/%E7%B6%B2%E9%9A%9B%E7%B6%B2%E8%B7%AF)及[局域网](https://zh.wikipedia.org/wiki/%E5%B1%80%E5%9F%9F%E7%BD%91)中，使用[虚拟终端机](https://zh.wikipedia.org/wiki/%E8%99%9B%E6%93%AC%E7%B5%82%E7%AB%AF%E6%A9%9F)的形式，提供双向、以文字字符串为主的[命令行接口](https://zh.wikipedia.org/wiki/%E5%91%BD%E4%BB%A4%E5%88%97%E4%BB%8B%E9%9D%A2)交互功能。

```
wikipedia:
Telnet is a protocol used on the Internet or local area network to provide a bidirectional interactive text-oriented communication facility using a virtual terminal connection. User data is interspersed in-band with Telnet control information in an 8-bit byte oriented data connection over the Transmission Control Protocol (TCP).
```

通常也用来测试远程端口是否开启。

## 启用

在 windows 7下，默认是没有启用telnet的。需要通过

```
控制面板->程序和功能->打开或关闭Windows功能->勾选Telnet客户端和Telnet服务端
```

来启用。

## 示例

```bash
telnet 47.98.108.25 5432
```

