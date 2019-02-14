---
title: redis��Ⱥ��ĵ�
tags:
  - tool
categories:
  - linux
date: 2019-02-14 01:00:00
---
> redis��Ⱥ��ĵ�
<!-- more -->

## ���밲װ
```
tar -xzvf redis-4.0.2.tar.gz 
cd redis-4.0.2
yum install gcc
make MALLOC=libc
make install
```
## ����redis�ڵ�,��̨������
```
==��81
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

==��82
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


==��84
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
## ���������ڵ㣬��̨������
```
redis-server /home/redis_cluster/7000/redis.conf
redis-server /home/redis_cluster/7001/redis.conf
```
## ������Ⱥ��ֻ��redis-trib.rbִ�ж˰�װ���³���
```
redis-3.2.2.gem�ļ�������84����������������ֻ��84��ִ�С�ע�������ruby��redis�����汾���ú�redis����˰汾һ�£�ruby��redis�����汾��redis-3.2.2��redis����˵İ汾��redis-4.0.2
yum install ruby ruby-devel rubygems rpm-build
gem install redis-3.2.2.gem
./redis-trib.rb create --replicas 1 10.24.70.81:7000 10.24.70.81:7001 10.24.70.82:7000 10.24.70.82:7001 10.24.70.84:7000 10.24.70.84:7001
```
## ��ά��
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