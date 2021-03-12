---
title: crontab+shell监控http服务并重启
toc: false
date: 2021-03-10 16:17:00
description: ...
tags:
- Linux
- Shell
---

今天，同事告诉我一个老(没)项(人)目(管)的HTTP服务经常挂掉，于是用最简单的办法，写了一个Shell脚本，进行HTTP请求，如果返回的状态码不对，就重启这个服务。

```shell
#!/bin/bash
URL=http://127.0.0.1:9085
RESTART_SCRIPT_PATH=/toplinker/topibd-http/topibd-http-server-b2b-9085/bin/restart

status_code=$(curl --write-out %{http_code} --silent --output /dev/null  "$URL")
echo "status_code: $status_code"
if [[ "$status_code" -ge 200 ]] && [[ "$status_code" -lt 400 ]]; then
  echo "online"
else
  echo "offline"
  sh "$RESTART_SCRIPT_PATH"
fi
```

然后用crontab设置每分钟跑一次脚本

```shell
$ crontab -l
* * * * * sh /toplinker/topibd-http/topibd-http-server-b2b-9085/monitor/m.sh
```



