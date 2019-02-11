---
title: Oracle安装文档
tags:
  - oracle
categories:
  - db
date: 2019-01-26 01:00:00

---

> oracle11.2.0.4 + centos 7.5安装文档

<!-- more -->

软件准备
--------
Oracle版本是11.2.0.4
主机系统环境
------------
### 示例环境
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/38af0a72d5fb814cd7c8715c819ed9cf.png)
### 修改系统标识
`$> echo "Red Hat Enterprise Linux Server release 7.0 (Maipo)" >/etc/redhat-release`
### 修改主机名
`$> hostnamectl set-hostname oracle-server`
### 修改系统内核参数
```
$> echo "
fs.aio-max-nr = 1048576
fs.file-max = 6815744
#单位是页
kernel.shmall = 7864320
kernel.shmmax = 132212254720
kernel.shmmni = 4096
#semaphores: semmsl, semmns, semopm, semmni
#SEMMSL==>processes+10 SEMMNS===>SEMMNI*SEMMSL SEMOPM====>系统调用允许的信号量最大个数至少100
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
$> sysctl –p
```
### 修改hosts文件
```
$>echo "# Oracle-Server
192.168.106.111 oracle-server
" >> /etc/hosts
```
### 修改操作系统资源限制
```
$>echo "#ORACLE
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 4096
oracle hard nofile 65536
oracle soft stack 10240" >> /etc/security/limits.conf
$>echo "session required pam_limits.so" >> /etc/pam.d/login
$>#如果存在文件/etc/security/limits.d/20-nproc.conf那么做如下修改
$>vi /etc/security/limits.d/20-nproc.conf
#把下面这句
* soft nproc 1024
#修改为
* - nproc 16384
```
### 创建用户和组
```
$>groupadd -g 54321 oinstall
$>groupadd -g 54322 dba
$>groupadd -g 54323 oper
$>useradd -u 54322 -g oinstall -G dba,oper oracle
$>passwd oracle
```
### 修改oracle用户环境变量
>在/home/oracle/.bash_profile末尾添加

```
#Oracle Settings
export TMP=/tmp
export TMPDIR=$TMP
export ORACLE_HOSTNAME=oracle-server
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE /db11204
export ORACLE_SID=ORCL
export BASE_PATH=/usr/sbin:$PATH
export PATH=$ORACLE_HOME/bin:$BASE_PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
```
### 创建目录及授权
```
$>mkdir -p /u01/app/oracle
$>mkdir -p /u01/app/oracle/db11204
$>chown oracle:oinstall /u01/app
$>chown -R oracle:oinstall /u01/app/oracle
$>chmod -R 0775 /u01
```
### 安装oracle所需的软件环境
```
$>rpm -Uvh compat-libstdc++-33-3.2.3-72.el7.x86_64.rpm
$>rpm -Uvh ksh-20120801-137.el7.x86_64.rpm
$>rpm -ivh libaio-devel-0.3.109-13.el7.x86_64.rpm
$>rpm -Uvh unixODBC-2.3.1-11.el7.x86_64.rpm
$>rpm -Uvh unixODBC-devel-2.3.1-11.el7.x86_64.rpm
$>rpm -ivh zlib-devel-1.2.7-17.el7.x86_64.rpm
$>rpm -ivh elfutils-libelf-devel-0.170-4.el7.x86_64.rpm
$>rpm -ivh libXmu-1.1.2-2.el7.x86_64.rpm
$>rpm -Uvh libXaw-1.0.13-4.el7.x86_64.rpm
$>rpm -Uvh xterm-295-3.el7.x86_64.rpm
$>rpm -Uvh perl-Switch-2.16-7.el7.noarch.rpm
```
>注意：上面是本文档环境实际执行的，如果linux操作系统有出入，则有两种方式获取安装包，其一，去系统安装盘中寻找对应版本的软件，其二，用yum下载，例如ksh-20120801-137.el7.x86_64.rpm可以在linux中执行yum
–y install ksh即可。
### 设置selinux并使之生效
```
$>sed -i '/SELINUX=enforcing/s/enforcing/permissive/g' /etc/selinux/config
$>setenforce Permissive
```
### 关闭防火墙并取消开机启动
```
$>systemctl stop firewalld
$>systemctl disable firewalld
```
### 设置NTP时钟
```
$>systemctl stop ntpd
$>systemctl disable ntpd
$>mv /etc/ntp.conf /etc/ntp.conf.orig
```
Oracle安装
----------

执行./runInstaller

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/8648eb24e32f99e7aca5c7f07caf5ff1.png)

选择不更新，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/c1ac5f05f5335f3cae0c348b5a417ca4.png)

确定

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/be2e87a8b94516b5a3114cd7890ade8c.png)

选择仅安装软件，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/729535785378935c8224af2949e32774.png)

选择单实例，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/a1666c25e1f4215d4d19864283df33f6.png)

默认，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/8f17a8f328d71efddb948a86193d06b9.png)

选择企业版，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/52b0d90f783e0876fadd16a9373b0d9c.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/ea1b2d772c9ccf3219e74f46540b2619.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/dcb93cdf3f63ab7dcd04e48075d004d4.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/51c3acaa721259649fa9d3bc8f228895.png)

>说明：因为文档中安装的是64位oracle，所以上图依赖环境中如果检测到有32位软件未按照时，可以忽略。除了上面说的，还有其他依赖检查失败，并且安装步骤是按照本文档执行的，那么需要联系DBA解决这个问题。

选择忽略所有，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/8bfbfe52895fd7b808fb59832a777d31.png)

完成

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/e9776a1b1aafc7341dc648d82e0ea4fd.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/bb76efedbef27d89c3f01fb08b347605.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/e1abef691028b5f6c1a0a1e3512b0341.png)

用root账户执行上面脚本，点完成

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/03ca5ef7c40f221da405b247c57c8ffb.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/34374713818d831761b03cc720598b56.png)

Oracle监听配置
--------------

执行netca

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/c55ac08666992251a903f44b1656959d.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/520ce2bb9a399c6978966b0fa4506873.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/632cbdeeee59a7eae16860470d1b27f0.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/bfb4ea1200def1165037586694d8dc58.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/b8520c9225bf64b9f20193abb8801ca2.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/d0a74ce6e52183e75dbe76ec8af358a1.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/d8bbd22b2a716b62e6e93a77bfe2a3ca.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/f64f4eb3a1830778b7cc5aa8abd30813.png)

点完成

Oracle数据库配置
----------------

执行dbca

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/add043123b071bb3dce44725f4e44704.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/90454c0fe8afbc3c4ec7c5591e8639e0.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/5b46bdce515dca7d38f4ba3683d6888a.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/b622361a804e02135b54467cf87a4381.png)

指定实例名，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/3cd232ed89eca28cbd404db1359f028a.png)

默认，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/1c37b26e3199013a9a43f7bc8e5b6d9d.png)

所有账号使用同一个密码，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/4c492b3fe795666bf5d19f5a66fdb108.png)

确定

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/1797408e37336a017e0df5d4190953a5.png)

默认下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/e30d0d1e83e5ce99f7abe993bde841da.png)

闪回区至少设置10G，下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/6cb0ee8cb2d924cd3213945e1bd7df1c.png)

默认下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/457c22f538bacf7760e2365799ff3f29.png)

内存设置40%

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/0dd008b83fd27b06efb85dda2563e42e.png)

进程设置1500

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/0856628e3c2f837ae1f20dbce08f51e6.png)

字符集选择AL32UTF8

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/12dc8ffd360557adacd1647a981b3528.png)

下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/928d07c37e387b4832990cfb05db24bd.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/781e7ad2cd7687fc9990fd28a5a4251e.png)

3个redo文件大小设置1G到10G之间，不确定的话，默认3个文档都设置为1G。下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/8db707ab0be9692bce8e1a589fa3f74e.png)

完成

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/d94f8d4f86c0abdc13483a255afbd371.png)

确定

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/47920725615af4f0f59157ea7127c75b.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/e38ee84fcab229f25a0d4b1ccc6a47ef.png)

等待安装完成
