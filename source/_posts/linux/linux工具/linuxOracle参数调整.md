---
title: linux参数配置
tags:
  - tool
categories:
  - linux
date: 2019-02-20 05:00:00
---
> linux参数配置
<!-- more -->
## linux配置

```
[root@xxx ~]# vi /etc/security/limits.conf
*             soft    nproc           2065651
*             hard    nproc           2065651
*             soft    nofile          65535
*             hard    nofile          65535

[root@xxx ~]# vi /etc/sysctl.conf
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 1024    65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 500
vm.overcommit_memory = 2
vm.overcommit_ratio= 85
vm.zone_reclaim_mode=0
fs.file-max=6553560
[root@xxx ~]# sysctl -p
```
## oracle 64G服务器例子
```
[root@xxx ~]# echo "
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 7864320
kernel.shmmax = 132212254720
kernel.shmmni = 4096
#semaphores: semmsl, semmns, semopm, semmni
kernel.sem = 250 32000 100 128
net.core.rmem_default=262144
net.core.rmem_max=16777216
net.core.wmem_default=262144
net.core.wmem_max=16777216
net.core.somaxconn=4096
net.core.netdev_max_backlog=262144
net.ipv4.tcp_max_syn_backlog=262144
net.ipv4.tcp_max_tw_buckets=10000
net.ipv4.ip_local_port_range = 9000 65500
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 3600
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_mem = 786432 1048576 1572864
vm.swappiness = 0
vm.panic_on_oom = 1
kernel.randomize_va_space=0" >>/etc/sysctl.conf
```
## mysql例子
```
vm.swappiness=1
net.core.somaxconn = 65535
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 1024    65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 500
vm.overcommit_memory = 2
vm.overcommit_ratio= 85
vm.zone_reclaim_mode=0
fs.file-max=6553560 
```
## 其他
```
readahead设置（谨慎）
[root@xxx ~]#blockdev --setra 16 /dev/sdb1
noatime set
[root@xxx ~]# vi /etc/fstab
/dev/mapper/vg_data-lv_data /opt                   ext4    rw,noatime        1 2
```