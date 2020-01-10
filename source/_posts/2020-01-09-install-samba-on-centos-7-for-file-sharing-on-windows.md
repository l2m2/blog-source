---
title: CentOS 7安装Samba与Windows共享文件夹
toc: true
date: 2020-01-09 17:47:02
description: 现在你是Linux菜逼，以后你还是。~_~
tags:
- CentOS
---

## 安装Samba

```bash
[root@192 ~]# yum install samba samba-client samba-common
```

设置防火墙

```bash
[root@192 ~]# firewall-cmd --permanent --zone=public --add-service=samba
success
[root@192 ~]# firewall-cmd --reload
success
```

## 配置Samba

### 查看工作站域

注意：Windows机器必须和CentOS 7服务器在同一个工作站域 内。*

在Windows上，可以通过以下命令查看工作站域 。

```bash
C:\Users\Administrator.CDPC011>net config workstation
计算机名                     \\CDPC011
计算机全名                   cdpc011
用户名                       Administrator
工作站正运行于
        NetBT_Tcpip_{B9A3B273-18A2-40C2-B331-D3BCACBFEE26} (00090FAA0001)
        NetBT_Tcpip_{F1B85F1A-D3DB-4C53-A90D-5DEEF8CB78A3} (002324E0954E)
        NetBT_Tcpip_{2EC6ACA8-14C5-425F-8642-B5D8A64840B1} (005056C00001)
        NetBT_Tcpip_{1D06B3C3-1FE8-4289-ABAB-41573366F053} (005056C00008)
软件版本                     Windows 7 Ultimate
工作站域                     WORKGROUP
登录域                       CDPC011
```

在我的环境下，工作站域是`WORKGROUP`。

### smb.conf

安全起见，自定义配置前先备份原来的配置。

```bash
[root@192 ~]# cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

### 公共文件夹共享

先创建一个匿名用户共享的文件夹

```bash
[root@192 home]# mkdir -p /home/data/samba/anonymous
[root@192 home]# chmod -R 0777 /home/data/samba/anonymous/
[root@192 home]# chown -R nobody:nobody /home/data/samba/anonymous
```

更改匿名共享文件夹的SELinux安全设置

```bash
[root@192 home]# chcon -t samba_share_t /home/data/samba/anonymous
```

然后，更改Samba的配置

```bash
[root@192 ~]# cat /etc/samba/smb.conf
[global]
	workgroup = WORKGROUP
	security = user
	map to guest = bad user
[Anonymous]
	comment = Anonymous File Server Share
	path = /home/data/samba/anonymous
	browsable = yes
	writeable = yes
	guest ok  = yes
	read only = no
	public = yes
```

启动服务且设置开机自启

```bash
[root@192 system]# systemctl enable smb
Created symlink from /etc/systemd/system/multi-user.target.wants/smb.service to /usr/lib/systemd/system/smb.service.
[root@192 system]# systemctl enable nmb
Created symlink from /etc/systemd/system/multi-user.target.wants/nmb.service to /usr/lib/systemd/system/nmb.service.
[root@192 system]# systemctl restart smb
[root@192 system]# systemctl restart nmb
```

### 私密文件夹共享

添加一个samba组，再添加用户到该组内

```bash
[root@192 ~]# groupadd sambagroup
[root@192 ~]# usermod leon -aG sambagroup
[root@192 ~]# smbpasswd -a leon
New SMB password:
Retype new SMB password:
Added user leon.
```

再创建一个私密文件夹，并设置权限

```bash
[root@192 ~]# mkdir -p /home/data/samba/secure
[root@192 ~]# chmod -R 0770 /home/data/samba/secure
[root@192 ~]# chown -R root:sambagroup /home/data/samba/secure
[root@192 ~]# chcon -t samba_share_t /home/data/samba/secure
```

然后再修改smb.conf

```bash
[root@192 samba]# cat /etc/samba/smb.conf
[global]
	workgroup = WORKGROUP
	security = user
	map to guest = bad user
[Anonymous]
	comment = Anonymous File Server Share
	path = /home/data/samba/anonymous
	browsable = yes
	writeable = yes
	guest ok  = yes
	read only = no
	public = yes
[Secure]
	comment = Secure File Server Share
	path = /home/data/samba/secure
	valid users = @sambagroup
	guest ok = no
	writable = yes
	browsable = yes
```

再重启服务

```bash
[root@192 ~]# systemctl restart smb
[root@192 ~]# systemctl restart nmb
```

测试完毕，打完收工！

## Reference

-  https://www.tecmint.com/install-samba4-on-centos-7-for-file-sharing-on-windows/ 

-  https://www.samba.org/samba/docs/4.7/man-html/smb.conf.5.html 

