---
title: redis主从+自动切换
tags:
  - tool
categories:
  - linux
date: 2019-02-21 05:00:00
---
> redis主从+自动切换
> <!-- more -->

## 正式环境redis主从配置
### 配置
```
[root@localhost ~]# mkdir /home/redis_copy
[root@localhost ~]# mkdir /home/redis_copy/6500
[root@localhost ~]# vi /home/redis_copy/6500/redis.conf
#master
port 6500
bind 10.24.70.84
daemonize yes
logfile redis6500.log
pidfile 6500.pid
#cluster-enabled yes
#cluster-config-file nodes_7000.conf
#cluster-node-timeout 15000
#appendonly yes
masterauth 123456
requirepass 123456




timeout 0
rdbcompression yes
dbfilename redis.rdb
dir /home/redis_copy/6500
maxmemory 50gb
maxmemory-policy noeviction
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

vm-enabled no

#slave
port 6500
bind 10.24.70.81
daemonize yes
logfile redis6500.log
pidfile 6500.pid
#cluster-enabled yes
#cluster-config-file nodes_7000.conf
#cluster-node-timeout 15000
#appendonly yes
masterauth 123456
requirepass 123456
slaveof 10.24.70.84 6500
slave-read-only yes
slave-priority 100


timeout 0
rdbcompression yes
dbfilename redis.rdb
dir /home/redis_copy/6500
maxmemory 50gb
maxmemory-policy noeviction
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

vm-enabled no
```
### 启动
```
[root@localhost ~]# redis-server /home/redis_copy/6500/redis.conf
```
### 81上配置哨兵
```
[root@localhost ~]# mkdir /home/redis_copy/6600
[root@localhost ~]# vi /home/redis_copy/6600/sentinel.conf

port 6600
daemonize yes
protected-mode no
logfile sentinel.log
dir /home/redis_copy/6600
sentinel monitor mymaster 10.24.70.84 6500 1
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 18000
sentinel auth-pass mymaster 123456
sentinel parallel-syncs mymaster 1
```
### 启动哨兵
```
[root@localhost ~]# redis-sentinel /home/redis_copy/6600/sentinel.conf
```

