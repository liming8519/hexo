---
title: redis集群搭建文档
tags:
  - tool
categories:
  - linux
date: 2019-02-14 01:00:00
---
> redis集群搭建文档
<!-- more -->

## 编译安装
```
tar -xzvf redis-4.0.2.tar.gz 
cd redis-4.0.2
yum install gcc
make MALLOC=libc
make install
```
## 创建redis节点,三台服务器
```
==》81
mkdir /home/redis_cluster
cd /home/redis_cluster
mkdir 7000 7001
vi 7000/redis.conf
port 7000
bind 10.24.70.81
daemonize yes
pidfile /home/redis_cluster/7000.pid
cluster-enabled yes
cluster-config-file nodes_7000.conf
cluster-node-timeout 15000
appendonly yes


vi 7001/redis.conf
port 7001
bind 10.24.70.81
daemonize yes
pidfile /home/redis_cluster/7001.pid
cluster-enabled yes
cluster-config-file nodes_7001.conf
cluster-node-timeout 15000
appendonly yes

==》82
mkdir /home/redis_cluster
cd /home/redis_cluster
mkdir 7000 7001
vi 7000/redis.conf
port 7000
bind 10.24.70.82
daemonize yes
pidfile /home/redis_cluster/7000.pid
cluster-enabled yes
cluster-config-file nodes_7000.conf
cluster-node-timeout 15000
appendonly yes


vi 7001/redis.conf
port 7001
bind 10.24.70.82
daemonize yes
pidfile /home/redis_cluster/7001.pid
cluster-enabled yes
cluster-config-file nodes_7001.conf
cluster-node-timeout 15000
appendonly yes


==》84
mkdir /home/redis_cluster
cd /home/redis_cluster
mkdir 7000 7001
vi 7000/redis.conf
port 7000
bind 10.24.70.84
daemonize yes
pidfile /home/redis_cluster/7000.pid
cluster-enabled yes
cluster-config-file nodes_7000.conf
cluster-node-timeout 15000
appendonly yes


vi 7001/redis.conf
port 7001
bind 10.24.70.84
daemonize yes
pidfile /home/redis_cluster/7001.pid
cluster-enabled yes
cluster-config-file nodes_7001.conf
cluster-node-timeout 15000
appendonly yes
```
## 启动各个节点，三台服务器
```
redis-server /home/redis_cluster/7000/redis.conf
redis-server /home/redis_cluster/7001/redis.conf
```
## 创建集群，只在redis-trib.rb执行端安装以下程序
```
redis-3.2.2.gem文件拷贝到84服务器，以下命令只在84上执行。注意这里的ruby的redis驱动版本不用和redis服务端版本一致，ruby的redis驱动版本是redis-3.2.2，redis服务端的版本是redis-4.0.2
yum install ruby ruby-devel rubygems rpm-build
gem install redis-3.2.2.gem
./redis-trib.rb create --replicas 1 10.24.70.81:7000 10.24.70.81:7001 10.24.70.82:7000 10.24.70.82:7001 10.24.70.84:7000 10.24.70.84:7001
```
## 简单维护
```
./redis-trib.rb check 10.24.70.81:7000
cluster info
cluster nodes
cluster meet ip port
cluster forget nodeid
cluster replicate nodeid
cluster saveconfig
cluster addslots slot
cluster delsolts slot
cluster flushslots
cluster setslot slot node nodeid
cluster setslot slot migrating nodeis
cluster setslot slot importing nodeid
cluster setslot slot stable 
cluster keyslot key
cluster countkeysinslot slot
cluster getkeysinslot slot count
cluster slaves nodeid
```