---
title: CentOS 7系统信息查看
toc: false
date: 2020-07-01 15:04:00
description: 备忘
tags:
- CentOS
---

- 操作系统版本

  ```bash
  $ cat /etc/redhat-release 
  CentOS Linux release 7.3.1611 (Core) 
  $ cat /etc/centos-release
  CentOS Linux release 7.3.1611 (Core) 
  ```

- 查看CPU型号

  ```bash
  $ cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
        8  Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.50GHz
  ```

- 系统进程

  ```bash
  $ ps aux
  USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root         1  0.0  0.0 191248  3192 ?        Ss    2019 142:31 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
  root         2  0.0  0.0      0     0 ?        S     2019   0:08 [kthreadd]
  root         3  0.0  0.0      0     0 ?        S     2019   4:24 [ksoftirqd/0]
  root         5  0.0  0.0      0     0 ?        S<    2019   0:00 [kworker/0:0H]
  root         7  0.0  0.0      0     0 ?        S     2019   4:44 [migration/0]
  root         8  0.0  0.0      0     0 ?        S     2019   0:00 [rcu_bh]
  root         9  0.1  0.0      0     0 ?        S     2019 657:43 [rcu_sched]
  ```

  `ps`是Process Status的缩写。

  `USER`: 进程所有者的用户名。

  `PID`: 进程ID。

  `%CPU`: 进程的CPU占用率。

  `%MEM`: 进程的内存占用率。

  `VSZ`: 进程使用的虚拟内存的大小（单位：KB）。

  `RSS`: 未交换的物理内存大小（单位：KB）。

  `TTY`: 与进程关联的终端。

  `STAT`: 进程的状态。

  `TIME`: 使用的CPU累计时间。

  `COMMAND`: 执行的命令。

- 内存使用情况

  ```bash
  $ free -h
                total        used        free      shared  buff/cache   available
  Mem:            15G         10G        334M        886M        5.0G        4.1G
  Swap:            0B          0B          0B
  
  ```

- 列出所有可用块设备

  ```bash
  $ lsblk
  NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
  sr0     11:0    1 1024M  0 rom  
  vda    253:0    0   50G  0 disk 
  └─vda1 253:1    0   50G  0 part /
  vdb    253:16   0  200G  0 disk 
  └─vdb1 253:17   0  200G  0 part /data
  ```

  `NAME`: 设备名称。

  `MAJ:MIN`: 主要和次要设备号。

  `RM`: 是否可移动设备。

  `SIZE`: 容量。

  `RO`: 是否只读。

  `TYPE`: 类型。vda和vdb是磁盘，sro是只读存储ROM，part是分区。

  `MOUNTPOINT`: 设备的挂载点。

- 详细的磁盘使用情况

  ```bash
  $ df -h
  Filesystem      Size  Used Avail Use% Mounted on
  /dev/vda1        50G   49G     0 100% /
  devtmpfs        7.8G     0  7.8G   0% /dev
  tmpfs           7.8G     0  7.8G   0% /dev/shm
  tmpfs           7.8G  1.7M  7.8G   1% /run
  tmpfs           7.8G     0  7.8G   0% /sys/fs/cgroup
  /dev/vdb1       197G  159G   29G  85% /data
  tmpfs           1.6G  8.0K  1.6G   1% /run/user/42
  tmpfs           1.6G     0  1.6G   0% /run/user/0
  overlay         197G  159G   29G  85% /data/docker/overlay/89f7cdc682c9b0f9db9dba8bac4f9c54dd18d2b149f12da1668a73cb237cc5f6/merged
  overlay         197G  159G   29G  85% /data/docker/overlay/2d6102ef258a1fea985cc06ea39d2492de7063697eb54378c0e0af4e0ee1b5e8/merged
  overlay         197G  159G   29G  85% /data/docker/overlay/f7dbd5a6f6a1eb1721edfca1d7320e2dc7b7c8792a0c69203bb806c56ee6a435/merged
  ```

  `df`是磁盘可用情况(Disk Free)的缩写。

  `Filesystem`: 文件系统。

  `Size`: 大小。

  `Used`: 已用。

  `Avail`:  可用。

  `Use%`: 已用占比。

  `Mounted on`: 挂载点。

- 查看当前目录的子目录大小

  ```bash
  $ du -h --max-depth=1
  318M	./sincpm-server-9183-deleted
  623M	./data
  140M	./topnginx
  925M	./tjspm-server-9182
  20K	./svn
  52K	./data_robot
  395M	./topapi
  ```

  `du`是磁盘使用情况(Disk Usage)的缩写。

## Reference

- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide

