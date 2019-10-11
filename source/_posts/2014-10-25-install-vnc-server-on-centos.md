---
title: CentOS搭建VNC服务
toc: true
date: 2014-10-25 16:30:25
description: 通常使用VNC进行远程桌面访问
tags:
- CentOS
---
## 安装

1. 安装tigervnc-server

   ```sh
   yum install tigervnc-server
   ```

2. 配置密码，你可以同时配置一个view only的密码

   ```sh
   vncpasswd
   ```

3. 配置VNC

    ```sh
    cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
    ```
    
    编辑此文件如下：
    ```sh
    [Unit]
    Description=Remote desktop service (VNC)
    After=syslog.target network.target
    
    [Service]
    Type=forking
    # Clean any existing files in /tmp/.X11-unix environment
    ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
    ExecStart=/usr/sbin/runuser -l root -c "/usr/bin/vncserver %i -geometry 1280x1024"
    PIDFile=/root/.vnc/%H%i.pid
    ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
    
    [Install]
    WantedBy=multi-user.target
    ```

4. 启动VNC

    ```shell
    systemctl daemon-reload
    systemctl start vncserver@:1
    systemctl status vncserver@:1
    systemctl enable vncserver@:1
    ```


5. 为了允许外部VNC客户端连接到CentOS中的VNC服务器，您需要确保允许正确的VNC开放端口通过防火墙。如果只启动了一个VNC服务器实例，您只需打开第一个分配的VNC端口：5901 / TCP，通过发出以下命令在运行时应用防火墙配置。

   ```
   firewall-cmd --add-port=5901/tcp --permanent
   firewall-cmd --reload
   ```

## 卸载

```sh
yum remove tigervnc-server
```

## 常见问题

- 在Windows端用VNC Viewer连接测试， 若出现蓝屏

  ```
  yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
  ```

- 启动服务时报错`Job for vncserver@:1.service failed because a configured resource limit was exceeded. See "systemctl status vncserver@:1.service" and "journalctl -xe" for details.`

  ```
  rm -f /tmp/.X11-unix/X1
  ```

  <https://serverfault.com/questions/925061/unable-to-start-vnc-server-configured-resource-limit-was-exceeded>

## Reference

- [How to Install and Configure VNC Server in CentOS 7](https://www.tecmint.com/install-and-configure-vnc-server-in-centos-7/)