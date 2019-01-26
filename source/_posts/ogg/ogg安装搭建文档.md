---
title: ogg安装和搭建文档
tags:
  - ogg
categories:
  - tool
date: 2019-01-26 15:00:00
---

>ogg的安装和搭建内容
<!-- more -->

## 安装ogg软件
### 建立ogg软件目录
```
[root@localhost oracle]# su - oracle
[oracle@localhost ~]$ cd /home/oracle/app/oracle
[oracle@localhost oracle]$ mkdir ogg
```
### 复制ogg软件到目标机器
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/0ca089f3b044e3c8ebb70034da8241ae.png)
### 解压软件
```
[oracle@localhost ogg]$ pwd
/home/oracle/app/oracle/ogg
[oracle@localhost ogg]$ ls
122022_fbo_ggs_Linux_x64_shiphome.zip
[oracle@localhost ogg]$ unzip 122022_fbo_ggs_Linux_x64_shiphome.zip
[oracle@localhost ogg]$ ls
122022_fbo_ggs_Linux_x64_shiphome.zip fbo_ggs_Linux_x64_shiphome
OGG-12.2.0.2-README.txt OGGCORE_12.2.0.2.2.pdf
[oracle@localhost ogg]$ rm OGG-12.2.0.2-README.txt OGGCORE_12.2.0.2.2.pdf
[oracle@localhost ogg]$ ls
122022_fbo_ggs_Linux_x64_shiphome.zip fbo_ggs_Linux_x64_shiphome
```

### 软件安装

打开xmanager进入目标机器，进入目录/home/oracle/app/oracle/ogg/fbo_ggs_Linux_x64_shiphome/Disk1，运行runInstaller安装。

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/ea6f4191a1a752f66325ef8c42d9234f.png)

选择oracle11g点击下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/86a029775b233dbe7736680f356b2b26.png)

配置后点击下一步

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/4cbb6ad0cc6267e4e094f2ffbed24fdf.png)

点击安装

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/f8f50b7ce0b573926a6daf82dce0d169.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/d4c1a6fbbcab268440c924304cf36667.png)

## 主从库初始化设置
>如果从库不做其他数据库的主库，那么supplemental log只需要在主库添加即可
```
[oracle@localhost ~]$ sqlplus / as sysdba
SQL*Plus: Release 11.2.0.1.0 Production on Thu Jul 5 12:40:45 2018
Copyright (c) 1982, 2009, Oracle. All rights reserved.
Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup mount
ORACLE instance started.
Total System Global Area 784998400 bytes
Fixed Size 2217464 bytes
Variable Size 486541832 bytes
Database Buffers 293601280 bytes
Redo Buffers 2637824 bytes
Database mounted.
SQL> alter database archivelog;
Database altered.
SQL> alter database add supplemental log data;
Database altered.
SQL> alter database open;
Database altered.
SQL> create user ogg identified by ogg;
User created.
SQL> grant connect,resource to ogg;
Grant succeeded.
SQL> grant execute on utl_file to ogg;
Grant succeeded.
SQL> grant select any dictionary,select any table to ogg;
Grant succeeded.
SQL> grant alter any table to ogg;
Grant succeeded.
SQL> grant flashback any table to ogg;
Grant succeeded.
SQL> grant execute on DBMS_FLASHBACK to ogg;
Grant succeeded.
SQL> grant dba to ogg;
Grant succeeded.
```
>注意：如果想简单的设置ogg的权限，那么ogg用户只赋dba权限既可。

### 数据源（主库）设置
>进入ogg即运行ggsci命令执行（执行./ggsci）：


```
> dblogin userid ogg password ogg
> add trandata smartecap_numone.tb*
> create subdirs
> edit params ./GLOBALS
_DISABLEFIX21427144 --远程访问
> edit params mgr
PORT 7809
DYNAMICPORTLIST 7810-7818,7820-7830 --远程访问端口
ACCESSRULE,PROG *,IPADDR *,ALLOW --远程访问
PURGEOLDEXTRACTS ./dirdat/st, USECHECKPOINTS, MINKEEPDAYS 7
>add trandata smartecap_numone.tb*
>info trandata smartecap_numone.tb* #查看是否设置传输数据成功
>add extract exts,tranlog ,begin now
>info all #查看所有进程
>add exttrail ./dirdat/st,extract exts,megabytes 1024
> edit params exts
EXTRACT exts
SETENV (NLS_LANG = "AMERICAN_AMERICA.ZHS16GBK")
SETENV (ORACLE_HOME = "/home/oracle/app/oracle/product/11.2.0/dbhome_1")
USERID ogg, PASSWORD ogg
REPORTCOUNT EVERY 1 MINUTES, RATE
DISCARDFILE ./dirrpt/exts.dsc,APPEND,MEGABYTES 1024
DBOPTIONS ALLOWUNUSEDCOLUMN
WARNLONGTRANS 2h,CHECKINTERVAL 350s
EXTTRAIL ./dirdat/st
FETCHOPTIONS NOUSESNAPSHOT
TRANLOGOPTIONS CONVERTUCS2CLOBS
IGNOREDELETES
TABLE smartecap_numone.tb*;
> add extract dpend,exttrailsource ./dirdat/st
> add rmttrail /home/oracle/app/ogg/dirdat/st ,extract dpend
> edit params dpend
EXTRACT dpend
SETENV (NLS_LANG = "AMERICAN_AMERICA.ZHS16GBK")
USERID ogg, PASSWORD ogg
PASSTHRU
RMTHOST 176.100.13.245, MGRPORT 7809, compress
RMTTRAIL /home/oracle/app/ogg/dirdat/st
TABLE smartecap_numone.tb*;
```

### 目标端（从库）设置
>进入ogg即运行ggsci命令执行（执行./ggsci）：

```
>create subdirs
>dblogin userid ogg,password ogg
>add checkpointtable ogg.checkpoint
>add replicat repnd,exttrail ./dirdat/st,checkpointtable ogg.checkpoint
>edit param repnd
REPLICAT repnd
SETENV (NLS_LANG = "AMERICAN_AMERICA.ZHS16GBK")
SETENV (ORACLE_HOME = "/home/oracle/app/oracle/product/11.2.0/dbhome_1")
USERID ogg, PASSWORD ogg
--HANDLECOLLISIONS
ASSUMETARGETDEFS
ddloptions report
MAP smartecap_numone.*, TARGET smartecap_numone.*;
```

## 备份主库的ddl语句并还原到从库

### ddl导出

可以使用工具导出ddl语句，这里使用toad，步骤如下：

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/6e777cb404cf6fa7e640f554689d89f7.png)

选择add并点击load rows

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/01fafae08aefbb91886abe825a6927a9.png)

选择需要导出的表

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/40a3af35b6908b0fac6ff857573bc523.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/611d6128ae38b1c8dec4a8cf8e4ebcd3.png)

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/4d4972afeb3828141c27690e2d0c8a39.png)

选择存储文件并点击导出

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/2f71c61b36f0f3454ae859f7107bc21e.png)

### 在从库创建用户并恢复主库的ddl
#### 创建用户
```
>CREATE USER SMARTECAP_NUMONE_XJ IDENTIFIED BY <password> ACCOUNT UNLOCK;
>GRANT CONNECT TO SMARTECAP_NUMONE_XJ;
>GRANT DBA TO SMARTECAP_NUMONE_XJ;
>GRANT RESOURCE TO SMARTECAP_NUMONE_XJ;
```
#### 恢复ddl语句
>使用toad登录从库的用户SMARTECAP_NUMONE_XJ，执行主库导出的ddl文件。

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/befe2282482863424bc057d4237441a0.png)

## 通过toad工具使主从数据同步，并同步
>在从库创建主库的dblink


```
create database link "source"
connect to SMARTECAP_NUMONE_XJ
identified by "123456"
using '192.168.106.174:1521/orcl';
```
>然后每张表都依次执行如下sql语句。为保证数据的一致性在导数过程中（ogg未同步前停止所有主库的应用）

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190126/7162ca279063e1ed5b5c623ab7cca4c6.png)

## ogg启动

### 首先在从库ggsci中执行
```
GGSCI (localhost.localdomain) 1> info all

Program Status Group Lag at Chkpt Time Since Chkpt
MANAGER RUNNING
REPLICAT STOPPED REPND 00:00:00 01:25:26

GGSCI (localhost.localdomain) 2> start repnd

Sending START request to MANAGER ...
REPLICAT REPND starting

GGSCI (localhost.localdomain) 3> info all

Program Status Group Lag at Chkpt Time Since Chkpt
MANAGER RUNNING
REPLICAT STARTING REPND 00:00:00 01:25:32

GGSCI (localhost.localdomain) 6> info all

Program Status Group Lag at Chkpt Time Since Chkpt
MANAGER RUNNING
REPLICAT RUNNING REPND 00:00:00 00:00:05
```
### 然后在主库ggsci中执行
```
GGSCI (oracle.domain) 23> info all

Program Status Group Lag at Chkpt Time Since Chkpt
MANAGER RUNNING
EXTRACT RUNNING DPEND 00:00:00 00:00:06
EXTRACT ABENDED DPENDD 00:00:00 01:25:16
EXTRACT RUNNING EXTS 00:00:00 00:00:00

GGSCI (oracle.domain) 24> start dpendd

Sending START request to MANAGER ...
EXTRACT DPENDD starting

GGSCI (oracle.domain) 25> info all

Program Status Group Lag at Chkpt Time Since Chkpt
MANAGER RUNNING
EXTRACT RUNNING DPEND 00:00:00 00:00:02
EXTRACT RUNNING DPENDD 00:00:00 01:25:22
EXTRACT RUNNING EXTS 00:00:00 00:00:06
```
