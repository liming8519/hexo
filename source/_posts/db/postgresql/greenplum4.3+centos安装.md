
---
title: greenplum4.3+centos安装
tags:
  - postgresql
categories:
  - db
date: 2019-04-22 01:00:00
---
> greenplum4.3+centos安装
<!-- more -->
　　
　　
### 初始化
```
[root@mdw ~]# vi /etc/hosts
172.16.69.130 mdw
172.16.69.131 sdw1
172.16.69.132 sdw2

[root@mdw ~]# vi /etc/sysctl.conf

kernel.shmmax = 500000000
kernel.shmmni = 4096
kernel.shmall = 4000000000
kernel.sem = 250 512000 100 2048
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.ip_forward = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.arp_filter = 1
net.ipv4.ip_local_port_range = 1025 65535
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.overcommit_memory = 2
```
### 创建用户
```
groupadd -g 530 gpadmin
useradd -g 530 -m -d /home/gpadmin -s /bin/bash gpadmin
passwd gpadmin
```
### 创建hostlist
```
su - gpadmin
mkdir -p /home/gpadmin/conf
vi /home/gpadmin/conf/hostlist
hadoopa
hadoopb
hadoopc

```
　　
### 创建seg_host
```
vi /home/gpadmin/conf/seg_host
hadoopb
hadoopc
```
### 配置ssh免密码登录
　　
```
su - gpadmin
source /usr/local/greenplum-db-4.3.26.0/greenplum_path.sh
 gpssh-exkeys -f /home/gpadmin/conf/hostlist
```
### 在Segment节点上安装Greenplum DB
　　
```
cd /usr/local
tar -cvf gdb.tar greenplum-db-4.3.26.0/
scp gdb.tar hadoopa:/usr/local
scp gdb.tar hadoopb:/usr/local
```
#### 每个节点执行解压
```
cd /usr/local
tar -xvf gdb.tar 
ln -s greenplum-db-4.3.26.0/ greenplum-db
```
### 初始化
```
cd conf
gpssh -f hostlist
cd
mkdir gpdata
cd gpdata
 mkdir gpmaster gpdatap1 gpdatap2 gpdatam1 gpdatam2
```
### 标记
```
[gpadmin@hadoopc conf]$ vi pgsh
export PGPORT=5432
export PGDATABASE=testDB
export MASTER_DATA_DIRECTORY=/home/gpadmin/gpdata/gpseg-1
source /usr/local/greenplum-db/greenplum_path.sh
```
　　
### 初始化，只配置主节点
```
vi /home/gpadmin/conf/gpinitsystem_config
ARRAY_NAME="Greenplum"
SEG_PREFIX=gpseg
PORT_BASE=33000
declare -a DATA_DIRECTORY=(/home/gpadmin/gpdata/gpdatap1  /home/gpadmin/gpdata/gpdatap2)
MASTER_HOSTNAME=hadoopc
MASTER_DIRECTORY=/home/gpadmin/gpdata/gpmaster
MASTER_PORT=5432
TRUSTED_SHELL=/usr/bin/ssh
MIRROR_PORT_BASE=43000
REPLICATION_PORT_BASE=34000
MIRROR_REPLICATION_PORT_BASE=44000
declare -a MIRROR_DATA_DIRECTORY=(/home/gpadmin/gpdata/gpdatam1 /home/gpadmin/gpdata/gpdatam2)
MACHINE_LIST_FILE=/home/gpadmin/conf/seg_host
```
### 初始化系统
```
gpinitsystem -c /home/gpadmin/conf/gpinitsystem_config -h /home/gpadmin/conf/hostlist
```