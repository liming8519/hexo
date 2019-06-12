---
title: greenplum5.18+centos扩展结点.
tags:

  - postgresql
categories:
  - db
date: 2019-06-12 01:00:00
---
> greenplum5.18+centos扩展结点.
<!-- more -->

> 扩展结点时，至少需要两个结点
## 初始化配置，参考安装文档
```
[root@mdw ~]# vi /etc/hosts
192.168.11.84 mdw
192.168.11.85 sdw1
192.168.11.86 sdw2
192.168.11.111 sdw3
192.168.11.118 hadoopb

[root@mdw ~]# vi /etc/sysctl.conf
kernel.shmmax = 500000000
kernel.shmmni = 4096
kernel.shmall = 4000000000
kernel.sem = 500 1024000 200 4096
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

[root@mdw ~]# vi /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072


groupadd -g 530 gpadmin
useradd -g 530 -m -d /home/gpadmin -s /bin/bash gpadmin
passwd gpadmin

mkdir /opt/greenplum
chown gpadmin:gpadmin /opt/greenplum
```

## 主节点上执行
```
$su - gpadmin
$cd conf/
$vi newhost 
sdw3
sdw4
$source pgsh##脚本中是环境变量
$cat pgsh
export PGPORT=5432
export PGDATABASE=mydb
export MASTER_DATA_DIRECTORY=/opt/greenplum/data/gpdata/gpmaster/gpseg-1
source /opt/greenplum/greenplum-db/greenplum_path.sh


$cat hostlist 
mdw
sdw1
sdw2
sdw3
sdw4
```
## 配置ssh面密码登录
```
gpssh-exkeys -f /home/gpadmin/conf/hostlist
```
## 在新节点上安装Greenplum DB
```
gpseginstall -f /home/gpadmin/conf/newhost
```
## 在两个新加的结点上分别创建目录
```
su - gpadmin
mkdir -p /opt/greenplum/data/gpdata
cd /opt/greenplum/data/gpdata
mkdir gpdatam{1,2}  gpdatap{1,2}
##初始化完成
```


## 配置文件input_file
```
gpexpand -f newhost
20190612:18:22:05:023132 gpexpand:mdw:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 5.18.0 build commit:6aec9959d367d46c6b4391eb9ffc82c735d20102'
20190612:18:22:05:023132 gpexpand:mdw:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.3.23 (Greenplum Database 5.18.0 build commit:6aec9959d367d46c6b4391eb9ffc82c735d20102) on x86_64-pc-linux-gnu, compiled by GCC gcc (GCC) 6.2.0, 64-bit compiled on Apr  3 2019 14:45:51'
20190612:18:22:05:023132 gpexpand:mdw:gpadmin-[INFO]:-Querying gpexpand schema for current expansion state

System Expansion is used to add segments to an existing GPDB array.
gpexpand did not detect a System Expansion that is in progress.

Before initiating a System Expansion, you need to provision and burn-in
the new hardware.  Please be sure to run gpcheckperf to make sure the
new hardware is working properly.

Please refer to the Admin Guide for more information.

Would you like to initiate a new System Expansion Yy|Nn (default=N):
> Y

You must now specify a mirroring strategy for the new hosts.  Spread mirroring places
a given hosts mirrored segments each on a separate host.  You must be 
adding more hosts than the number of segments per host to use this. 
Grouped mirroring places all of a given hosts segments on a single 
mirrored host.  You must be adding at least 2 hosts in order to use this.



What type of mirroring strategy would you like?
 spread|grouped (default=grouped):
> 这个选择默认

    By default, new hosts are configured with the same number of primary
    segments as existing hosts.  Optionally, you can increase the number
    of segments per host.
    
    For example, if existing hosts have two primary segments, entering a value
    of 2 will initialize two additional segments on existing hosts, and four
    segments on new hosts.  In addition, mirror segments will be added for
    these new primary segments if mirroring is enabled.


How many new primary segments per host do you want to add? (default=0):
> 这里选择默认

Generating configuration file...

20190612:18:22:11:023132 gpexpand:mdw:gpadmin-[INFO]:-Generating input file...

Input configuration files were written to 'gpexpand_inputfile_20190612_182211' and 'None'.
Please review the file and make sure that it is correct then re-run
with: gpexpand -i gpexpand_inputfile_20190612_182211 -D mydb
                
20190612:18:22:11:023132 gpexpand:mdw:gpadmin-[INFO]:-Exiting...
[gpadmin@mdw conf]$ ls
gpexpand_inputfile_20190612_182211  gpinitsystem_config  hostlist  newhost  pgsh  seg_host
[gpadmin@mdw conf]$ cat gpexpand_inputfile_20190612_182211 
hadoopb:hadoopb:50000:/opt/greenplum/data/gpdata/gpdatap1/gpseg4:10:4:p:51000
sdw3:sdw3:40000:/opt/greenplum/data/gpdata/gpdatam1/gpseg4:16:4:m:41000
hadoopb:hadoopb:50001:/opt/greenplum/data/gpdata/gpdatap2/gpseg5:11:5:p:51001
sdw3:sdw3:40001:/opt/greenplum/data/gpdata/gpdatam2/gpseg5:17:5:m:41001
sdw3:sdw3:50000:/opt/greenplum/data/gpdata/gpdatap1/gpseg6:12:6:p:51000
hadoopb:hadoopb:40000:/opt/greenplum/data/gpdata/gpdatam1/gpseg6:14:6:m:41000
sdw3:sdw3:50001:/opt/greenplum/data/gpdata/gpdatap2/gpseg7:13:7:p:51001
hadoopb:hadoopb:40001:/opt/greenplum/data/gpdata/gpdatam2/gpseg7:15:7:m:41001
```

## 扩展
```
gpexpand -i gpexpand_inputfile_20190612_193623
```
## 如果扩展失败则执行回滚操作
```
gpstart -m(或者gpstart -R)
gpexpand -r -D zhangyun_db##可以不带数据库
gpstart -a
```
