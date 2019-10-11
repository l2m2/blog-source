---
title: 用Python搭建一个简易的HTTP服务
toc: false
date: 2014-07-08 16:26:12
description: SimpleHTTPServer
tags:
- Python
- CentOS
---
本文描述的是在CentOS 7搭建的过程。
### 创建Unit文件

```bash
vi /etc/systemd/system/simplehttp.service
```

文件内容如下

```ini
[Unit]
Description=My Simple Service
After=network.target

[Service]
Type=simple
User=leon
WorkingDirectory=/home/leon/simplehttpserverdir
ExecStart=/usr/bin/python -m SimpleHTTPServer 8081
ExecStop=/bin/kill `/bin/ps aux | /bin/grep SimpleHTTPServer | /bin/grep -v grep | /usr/bin/awk '{ print $2 }'`
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

### 启动SimpleHttp.service

```bash
systemctl daemon-reload
systemctl restart simplehttp.service
```