---
title: greenplum5.18+centos安装
tags:

  - postgresql
categories:
  - db
date: 2019-04-22 01:00:00
---
> greenplum5.18+centos安装
<!-- more -->


### 初始化
```
[root@mdw ~]# vi /etc/hosts
172.16.69.130 hadoopa
172.16.69.131 hadoopb
172.16.69.132 hadoopc

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
### 安装目录创建
```
mkdir /usr/local/gpadmin
chown gpadmin:gpadmin /usr/local/gpadmin
```
### 创建hostlist,在主节点
```
su - gpadmin
mkdir -p /home/gpadmin/conf
vi /home/gpadmin/conf/hostlist
hadoopa
hadoopb
hadoopc

```

### 创建seg_host，在主节点
```
vi /home/gpadmin/conf/seg_host
hadoopb
hadoopa
```
### 安装greenplum
```
chown gpadmin:gpadmin greenplum-db-5.18.0-rhel7-x86_64.bin 
chmod +x greenplum-db-5.18.0-rhel7-x86_64.bin
./greenplum-db-5.18.0-rhel7-x86_64.bin 
...
I HAVE READ AND AGREE TO THE TERMS OF THE ABOVE PIVOTAL SOFTWARE
LICENSE AGREEMENT.


********************************************************************************
Do you accept the Pivotal Database license agreement? [yes|no]
********************************************************************************

yes
********************************************************************************
Provide the installation path for Greenplum Database or press ENTER to 
accept the default installation path: /usr/local/greenplum-db-5.18.0
********************************************************************************

/usr/local/gpadmin/greenplum-db-5.18.0
********************************************************************************
Install Greenplum Database into /usr/local/gpadmin/greenplum-db-5.18.0? [yes|no]
********************************************************************************

yes
********************************************************************************
/usr/local/gpadmin/greenplum-db-5.18.0 does not exist.
Create /usr/local/gpadmin/greenplum-db-5.18.0 ? [yes|no]
(Selecting no will exit the installer)
********************************************************************************

yes
```
### 配置ssh免密码登录

```
su - gpadmin
cd conf
source /usr/local/gpadmin/greenplum-db/greenplum_path.sh
gpssh-exkeys -f /home/gpadmin/conf/hostlist
```
### 在Segment节点上安装Greenplum DB

```
su - gpadmin
source /usr/local/gpadmin/greenplum-db/greenplum_path.sh
gpseginstall -f /home/gpadmin/conf/hostlist
```
#### 集群检查，并安装检查修改
```
gpcheck -f /home/gpadmin/conf/hostlist -m hadoopc
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[INFO]:-dedupe hostnames
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[INFO]:-Detected platform: Generic Linux Cluster
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[INFO]:-generate data on servers
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[INFO]:-copy data files from servers
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[INFO]:-delete remote tmp files
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[INFO]:-Using gpcheck config file: /usr/local/gpadmin/greenplum-db/./etc/gpcheck.cnf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(None): utility will not check all settings when run as non-root user
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): on device (sr0) IO scheduler 'cfq' does not match expected value 'deadline'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): /etc/sysctl.conf value for key 'kernel.sem' has value '250 512000 100 2048' and expects '500 1024000 200 4096'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): soft nofile not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): hard nproc not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): soft nproc not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): hard nofile not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/mapper/cl-root has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/mapper/cl-root is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/mapper/cl-root is missing the recommended mount option 'noatime'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/mapper/cl-home has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/mapper/cl-home is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/mapper/cl-home is missing the recommended mount option 'noatime'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/sda1 has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/sda1 is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopb): XFS filesystem on device /dev/sda1 is missing the recommended mount option 'noatime'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): on device (sr0) IO scheduler 'cfq' does not match expected value 'deadline'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): /etc/sysctl.conf value for key 'kernel.sem' has value '250 512000 100 2048' and expects '500 1024000 200 4096'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): soft nofile not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): hard nproc not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): soft nproc not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): hard nofile not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/mapper/cl-root has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/mapper/cl-root is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/mapper/cl-root is missing the recommended mount option 'noatime'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/mapper/cl-home has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/mapper/cl-home is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/mapper/cl-home is missing the recommended mount option 'noatime'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/sda1 has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/sda1 is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopc): XFS filesystem on device /dev/sda1 is missing the recommended mount option 'noatime'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): on device (sr0) IO scheduler 'cfq' does not match expected value 'deadline'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): /etc/sysctl.conf value for key 'kernel.sem' has value '250 512000 100 2048' and expects '500 1024000 200 4096'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): soft nofile not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): hard nproc not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): soft nproc not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): hard nofile not found in /etc/security/limits.conf
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/mapper/cl-root has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/mapper/cl-root is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/mapper/cl-root is missing the recommended mount option 'noatime'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/mapper/cl-home has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/mapper/cl-home is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/mapper/cl-home is missing the recommended mount option 'noatime'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/sda1 has 5 XFS mount options and 4 are expected
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/sda1 is missing the recommended mount option 'allocsize=16m'
20190423:17:18:03:019173 gpcheck:hadoopc:gpadmin-[ERROR]:-GPCHECK_ERROR host(hadoopa): XFS filesystem on device /dev/sda1 is missing the recommended mount option 'noatime'
```
### 初始化
```
su - gpadmin
cd conf
gpssh -f hostlist
cd
mkdir gpdata
cd gpdata
mkdir gpmaster gpdatap1 gpdatap2 gpdatam1 gpdatam2
#说明
Master和Master Standby上创建：mkdir /data/master
Segment上创建Primary Segment：mkdir /data/primary
Segment上创建Mirror Segment（非必须）：mkdir /data/mirror
```


### 初始化，只配置主节点
```
su - gpadmin
vi /home/gpadmin/conf/gpinitsystem_config
################################################
#### REQUIRED PARAMETERS
################################################

#### Name of this Greenplum system enclosed in quotes.
ARRAY_NAME="Greenplum"
#### Naming convention for utility-generated data directories.
SEG_PREFIX=gpseg
#### Base number by which primary segment port numbers
#### are calculated.
PORT_BASE=40000

#### File system location(s) where primary segment data directories
#### will be created. The number of locations in the list dictate
#### the number of primary segments that will get created per
#### physical host (if multiple addresses for a host are listed in
#### the hostfile, the number of segments will be spread evenly across
#### the specified interface addresses).

declare -a DATA_DIRECTORY=(/home/gpadmin/gpdata/gpdatap1  /home/gpadmin/gpdata/gpdatap2)
#### OS-configured hostname or IP address of the master host.
MASTER_HOSTNAME=hadoopc

#### File system location where the master data directory
#### will be created.
MASTER_DIRECTORY=/home/gpadmin/gpdata/gpmaster

#### Port number for the master instance.
MASTER_PORT=5432
#### Shell utility used to connect to remote hosts.
TRUSTED_SHELL=/usr/bin/ssh

#### Maximum log file segments between automatic WAL checkpoints.
CHECK_POINT_SEGMENTS=8

#### Default server-side character set encoding.
ENCODING=UNICODE

################################################
#### OPTIONAL MIRROR PARAMETERS
################################################
#### Base number by which mirror segment port numbers
#### are calculated.
MIRROR_PORT_BASE=50000

#### Base number by which primary file replication port
#### numbers are calculated.
REPLICATION_PORT_BASE=41000

#### Base number by which mirror file replication port
#### numbers are calculated.
MIRROR_REPLICATION_PORT_BASE=51000

#### File system location(s) where mirror segment data directories
#### will be created. The number of mirror locations must equal the
#### number of primary locations as specified in the
#### DATA_DIRECTORY parameter.
declare -a MIRROR_DATA_DIRECTORY=(/home/gpadmin/gpdata/gpdatam1 /home/gpadmin/gpdata/gpdatam2)
################################################
#### OTHER OPTIONAL PARAMETERS
################################################

#### Create a database of this name after initialization.
DATABASE_NAME=mydb
```
### 初始化系统
```
gpinitsystem -c /home/gpadmin/conf/gpinitsystem_config -h /home/gpadmin/conf/seg_host
```

### 标记
```
[gpadmin@hadoopc conf]$ vi pgsh
export PGPORT=5432
export PGDATABASE=mydb
export MASTER_DATA_DIRECTORY=/home/gpadmin/gpdata/gpmaster/gpseg-1
source /usr/local/gpadmin/greenplum-db/greenplum_path.sh
```

