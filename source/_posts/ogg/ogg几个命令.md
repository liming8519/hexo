---
title: ogg 几个命令
date: 2019-01-26 16:00:00
tags:
- ogg
categories:
- tool
---
>ogg相关的几个命令
<!-- more -->

## 查询相关
1. info all
 - 作用：查看整体的运行情况
 - 示例
```
GGSCI (gavinprod.com) 5> info all
Program     Status      Group       Lag at Chkpt  Time Since Chkpt
MANAGER     RUNNING                                           
EXTRACT     RUNNING     DMP2        842:45:01     00:00:09    
EXTRACT     RUNNING     EXT1        841:19:24     00:00:04    
EXTRACT     RUNNING     EXT2        529:12:40     00:00:00
```
2. view params process
 - 作用：查看进程的参数设置
 - 示例
```
GGSCI (gavinprod.com) 6> view params EXT1
extract ext1
userid ggate@gavinprod, password oracle
rmthost odellprod.com, mgrport 7809
rmttrail /opt/oracle/ggate/dirdat/lt
ddl include mapped objname sender.*;
table sender.*;
```
3. info process
 - 作用：查看进程的信息性，包括进程的状态、Checkpoint信息、延时等
 - 示例
```
GGSCI (gavinprod.com) 7> info EXT1
EXTRACT    EXT1      Last Started 2015-01-28 05:31   Status RUNNING
Checkpoint Lag       529:05:45 (updated 00:00:02 ago)
Log Read Checkpoint  Oracle Redo Logs
                     2015-01-06 04:27:03  Seqno 24, RBA 18464196
                     SCN 0.1440415 (1440415)
```
4. info process detail
 - 作用：查看更加详细的信息，包括所使用的trail文件、参数文件、报告文件、警告文件的位置
 - 示例
```
GGSCI (gavinprod.com) 8> info EXT1 detail
EXTRACT    EXT1      Last Started 2015-01-28 05:31   Status RUNNING
Checkpoint Lag       431:32:57 (updated 00:00:04 ago)
Log Read Checkpoint  Oracle Redo Logs
                     2015-01-10 06:00:18  Seqno 31, RBA 40079092
                     SCN 0.1553661 (1553661)
  Target Extract Trails:
  Remote Trail Name                                Seqno        RBA     Max MB
  /opt/oracle/ggate/dirdat/lt                          1       1105        100
  Extract Source                          Begin             End             
  /opt/oracle/flash_recovery_area/GAVINPROD/archivelog/2015_01_10/o1_mf_1_31_bc2d3p27_.arc  2014-12-24 04:12  2015-01-10 06:00
  /opt/oracle/oradata/gavinprod/redo01.log  2014-12-24 00:23  2014-12-24 04:12
  Not Available                           * Initialized *   2014-12-24 00:23
Current directory    /opt/oracle/ggate
Report file          /opt/oracle/ggate/dirrpt/EXT1.rpt
Parameter file       /opt/oracle/ggate/dirprm/ext1.prm
Checkpoint file      /opt/oracle/ggate/dirchk/EXT1.cpe
Process file         /opt/oracle/ggate/dirpcs/EXT1.pce
Stdout file          /opt/oracle/ggate/dirout/EXT1.out
Error log            /opt/oracle/ggate/ggserr.log
```
5. info process showch
 - 作用：查看详细的关于checkpoint的细心你想，用于查询GoldenGate进行处理过的事物记录
Extract进程的Recovery checkpoing，他标识源数据最早的未被处理事物，可以查到该事物的redo log位于哪个日志文件以及该日志文件的序列号，所有序列号比它大得日志文件都需保留
 -  示例
```
GGSCI (gavinprod.com) 9> info EXT1 showch
EXTRACT    EXT1      Last Started 2015-01-28 05:31   Status RUNNING
Checkpoint Lag       00:00:00 (updated 00:00:06 ago)
Log Read Checkpoint  Oracle Redo Logs
                     2015-01-28 05:33:32  Seqno 42, RBA 2548736
                     SCN 0.1750257 (1750257)
Current Checkpoint Detail:
Read Checkpoint #1
  Oracle Redo Log
  Startup Checkpoint (starting position in the data source):
    Thread #: 1
    Sequence #: 19
    RBA: 37158416
    Timestamp: 2014-12-24 04:12:21.000000
    SCN: 0.1341390 (1341390)
    Redo File: /opt/oracle/oradata/gavinprod/redo01.log
  Recovery Checkpoint (position of oldest unprocessed transaction in the data source):
    Thread #: 1
    Sequence #: 42
    RBA: 2504208
    Timestamp: 2015-01-28 05:33:32.000000
    SCN: 0.1750256 (1750256)
    Redo File: /opt/oracle/oradata/gavinprod/redo03.log
  Current Checkpoint (position of last record read in the data source):
    Thread #: 1
    Sequence #: 42
    RBA: 2548736
    Timestamp: 2015-01-28 05:33:32.000000
    SCN: 0.1750257 (1750257)
    Redo File: /opt/oracle/oradata/gavinprod/redo03.log
Write Checkpoint #1
  GGS Log Trail
  Current Checkpoint (current write position):
    Sequence #: 1
    RBA: 1105
    Timestamp: 2015-01-28 05:33:46.462178
    Extract Trail: /opt/oracle/ggate/dirdat/lt
CSN state information:
  CRC: A4-7E-18-EE
  Latest CSN: 1749736
  Latest TXN: 7.29.894
  Latest CSN of finished TXNs: 1749736
  Completed TXNs: 7.29.894
Header:
  Version = 2
  Record Source = A
  Type = 10
  # Input Checkpoints = 1
  # Output Checkpoints = 1
File Information:
  Block Size = 2048
  Max Blocks = 100
  Record Length = 2048
  Current Offset = 0
Configuration:
  Data Source = 3
  Transaction Integrity = 1
  Task Type = 0
Status:
  Start Time = 2015-01-28 05:31:39
  Last Update Time = 2015-01-28 05:33:46
  Stop Status = A
  Last Result = 400
```
6. lag process
 - 作用：查看详细的延时信息
 - 示例
```
GGSCI (gavinprod.com) 10> lag EXT1
Sending GETLAG request to EXTRACT EXT1 ...
Last record lag: 23 seconds.
At EOF, no more records to process.
```
7. stats
 - 作用：查看进程处理的记录数
 - 示例
```
GGSCI (gavinprod.com) 13> stats EXT1, total
Sending STATS request to EXTRACT EXT1 ...
Start of Statistics at 2015-01-28 05:37:59.
DDL replication statistics (for all trails):
*** Total statistics since extract started     ***
        Operations                                      2899.00
        Mapped operations                                  0.00
        Unmapped operations                             2809.00
        Other operations                                  90.00
        Excluded operations                             2899.00
Output to /opt/oracle/ggate/dirdat/lt:
Extracting from GGATE.GGS_MARKER to GGATE.GGS_MARKER:
*** Total statistics since 2015-01-28 05:31:45 ***
        No database operations have been performed.
End of Statistics.
```
8. view report process
 - 作用：查看对应的报告文件
 - 示例
```
GGSCI (gavinprod.com) 11> view report EXT1
/***********************************************************************
                 Oracle GoldenGate Capture for Oracle
    Version 11.2.1.0.1 OGGCORE_11.2.1.0.1_PLATFORMS_120423.0230_FBO
   Linux, x64, 64bit (optimized), Oracle 11g on Apr 23 2012 08:42:16
Copyright (C) 1995, 2012, Oracle and/or its affiliates. All rights reserved.
                    Starting at 2015-01-28 05:31:33
/***********************************************************************
Operating System Version:
Linux
Version #1 SMP Mon Nov 12 02:14:55 EST 2007, Release 2.6.18-53.el5
Node: gavinprod.com
Machine: x86_64
                         soft limit   hard limit
Address Space Size   :    unlimited    unlimited
Heap Size            :    unlimited    unlimited
File Size            :    unlimited    unlimited
CPU Time             :    unlimited    unlimited
Process id: 5097
Description:
/***********************************************************************
**            Running with the following parameters                  **
/***********************************************************************
2015-01-28 05:31:33  INFO    OGG-03035  Operating system character set identified as UTF-8. Locale: en_US, LC_ALL:.
extract ext1
userid ggate@gavinprod, password ******
2015-01-28 05:31:35  INFO    OGG-03500  WARNING: NLS_LANG environment variable does not match database character set, or not set.
Using database character set value of AL32UTF8.
rmthost odellprod.com, mgrport 7809
rmttrail /opt/oracle/ggate/dirdat/lt
```
9 view ggsevt
 - 作用：查看日志
 - 示例
```
GGSCI (gavinprod.com) 14> view ggsevt
2014-09-10 01:26:35  INFO    OGG-00987  Oracle GoldenGate Command Interpreter for Oracle:  GGSCI command (ggate): edit params mgr.
2014-09-10 01:27:16  INFO    OGG-00987  Oracle GoldenGate Command Interpreter for Oracle:  GGSCI command (ggate): edit params mgr.
2014-09-10 01:27:27  INFO    OGG-00987  Oracle GoldenGate Command Interpreter for Oracle:  GGSCI command (ggate): start manager.
2014-09-10 01:27:28  INFO    OGG-00983  Oracle GoldenGate Manager for Oracle, mgr.prm:  Manager started (port 7809).
2014-09-10 01:27:40  INFO    OGG-00987  Oracle GoldenGate Command Interpreter for Oracle:  GGSCI command (ggate): add extract ext1  tranlog, begin now.
2014-09-10 01:29:24  INFO    OGG-00987  Oracle GoldenGate Command Interpreter for Oracle:  GGSCI command (ggate): add exttrail2014-09-10 02:24:03
2014-09-10 02:24:03  INFO    OGG-00983  Oracle GoldenGate Manager for Oracle, mgr.prm:  Manager started (port 7809).
2014-09-10 02:24:07  INFO    OGG-00987  Oracle GoldenGate Command Interpreter for Oracle:  GGSCI command (ggate): start EXT1.
2014-09-10 02:24:07  INFO    OGG-00963  Oracle GoldenGate Manager for Oracle, mgr.prm:  Command received from GGSCI on host gavinprod.com
2014-09-10 02:24:07  INFO    OGG-00975  Oracle GoldenGate Manager for Oracle, mgr.prm:  EXTRACT EXT1 starting.
2014-09-10 02:24:08  INFO    OGG-00992  Oracle GoldenGate Capture for Oracle, ext1.prm:  EXTRACT EXT1 starting.
2014-09-10 02:24:08  INFO    OGG-03035  Oracle GoldenGate Capture for Oracle, ext1.prm:  Operating system character set identified as UTF-8.
2014-09-10 02:24:19  INFO    OGG-01635  Oracle GoldenGate Capture for Oracle, ext1.prm:  BOUNDED RECOVERY: reset to initial or altered checkpoint.
2014-09-10 02:24:19  INFO    OGG-01815  Oracle GoldenGate Capture for Oracle, ext1.prm:  Virtual Memory Facilities for: BR
    anon alloc: mmap(MAP_ANON)  anon free: munmap
    file alloc: mmap(MAP_SHARED)  file free: munmap
    target directories:
    /opt/oracle/ggate/BR/EXT1.
2014-09-10 02:24:19  INFO    OGG-01815  Oracle GoldenGate Capture for Oracle, ext1.prm:  Virtual Memory Facilities for: COM
    anon alloc: mmap(MAP_ANON)  anon free: munmap
    file alloc: mmap(MAP_SHARED)  file free: munmap
    target directories:
    /opt/oracle/ggate/dirtmp.
2014-09-10 02:24:20  WARNING OGG-01423  Oracle GoldenGate Capture for Oracle, ext1.prm:  No valid default archive log destination directory found
```

## extract相关
1. 修改rba,checkpoint
```
alter EXT_ALO,extseqno 17,extrba 9194144,thread 1
alter EXT_ALO,extseqno 10,extrba 5740032,thread 2
```
2. 修改recovery checkpoint
```
alter EXT_ALO,ioextseqno 17,ioextrba 9193488,thread 1
alter  EXT_ALO,ioextseqno 10,ioextrba 5739536,thread 2
```
3. info extract cust_ext, detail
>抽取进程的详细信息


4. info extract ext*, showch  
>抽取进程的检查点信息,注意checkpoint和recovery checkpoint


5. INFO TRANDATA user_name.table_names
>表的补充日志

6. 常用添加extract命令
```
ggsci> add extract ext1, tranlog, begin now
ggsci> add extract ext1, tranlog, begin now, threads 4
ggsci> add extract ext1, tranlog, begin now, passive
ggsci> add extract ext1, extseqno 111, begin now
ggsci> add extract ext1, extrba 567890, begin 2012-02-02 12:00:00
ggsci> add extract ext1, sourceistable
ggsci> add extract ext1, exttrailsource /oracle/gg11/dirdat/hr   --data pump
ggsci> add extract ext1, vam                        -- VAM - Vendor Access Module
ggsci> add extract ext1, vamtrailsource /ogg/dirdat/vt
ggsci> add extract ext1, rmthost host123, mgrport 7810, rmtname fin
```
7. status
```
ggsci> STATUS MANAGER    -- To determine whether or not the Manager process is running
ggsci> STATUS EXTRACT group_name [, TASKS | ALLPROCESSES]   -- To determine whether or not Extract is running
ggsci> status extract extr_hr
ggsci> status extract ext*, tasks
ggsci> status extract *ext*, allprocesses
ggsci> STATUS REPLICAT group_name [, TASKS | ALLPROCESSES]  -- To determine whether or not Replicat is running
ggsci> status replicat emp_rep
ggsci> status replicat cust_rep, allprocesses
ggsci> STATUS ER group_wildcard_specification   -- To check the status of multiple Extract and Replicat groups as a unit
ggsci> status er *EX*
Reading Program Database management system Vacuum Pump The Word Find email address free
ggsci> STATS EXTRACT group_name [, statistic] [, TABLE table] [, TOTALSONLY table_specification] [, REPORTFETCH | NOREPORTFETCH] [, REPORTRATE HR|MIN|SEC] [, ... ]  -- To display statistics for one or more Extract group
ggsci> stats ext_hr
ggsci> stats extract ext
ggsci> stats extract ext2 reportrate sec
ggsci> stats extract fin, total, daily
ggsci> stats extract fin, total, hourly, table acct, reportrate min, reset, reportfetch
```
8. send
```
ggsci> SEND MANAGER [CHILDSTATUS [DEBUG]] [GETPORTINFO [DETAIL]] [GETPURGEOLDEXTRACTS]   -- To retrieve the status of the active Manager process or to retrieve dynamic port information as configured in the Manager parameter file
ggsci> send manager childstatus
ggsci> send manager childstatus debug
ggsci> send manager getportinfo
ggsci> send manager getportinfo detail
ggsci> send manager getpurgeoldextracts
ggsci> SEND EXTRACT group_name,
	{ CACHEMGR {CACHESTATS | CACHEQUEUES | CACHEPOOL} | FORCESTOP | FORCETRANS id [THREAD n] [FORCE] | GETLAG | GETTCPSTATS | LOGEND | REPORT | ROLLOVER | SHOWTRANS [id] [THREAD n] [COUNT n] [DURATION duration_unit] [TABULAR] [FILE file_name [DETAIL]] | SKIPTRANS id [THREAD n] [FORCE] | STATUS | STOP | TLTRACE {DEBUG | OFF | level} [SIZELIMIT size] [DDLINCLUDE | DDL[ONLY]] [FILE] file_name | TRACE[2] {tracefile | OFF} | TRACEINIT | TRANLOGOPTIONS {PURGEORPHANEDTRANSACTIONS | NOPURGEORPHANEDTRANSACTIONS} | TRANLOGOPTIONS TRANSCLEANUPFREQUENCY minutes | VAMMESSAGE "Teradata_command" | VAMMESSAGE {ARSTATS | INCLUDELIST [filter] | EXCLUDELIST [filter]} | VAMMESSAGE OPENTRANS
	}     -- To communicate with a running Extract process
	Teradata_command = {"control:terminate" | "control:suspend" | "control:resume" | "control:copy database.table"
ggsci> send extract exthr status
ggsci> send extract extr, getlag
ggsci> send extract group_name tltrace file file_name ddlinclude
ggsci> send extract fin, rollover
ggsci> send extract fin  stop
ggsci> send extract fin, vammessage control:suspend
ggsci> send extract fin, tranlogoptions transcleanupfrequency 15
ggsci> send extract fin, showtrans count 10
ggsci> send extract fin, skiptrans 5.17.27634 thread 2
```
9. alter
```
ggsci> ALTER EXTRACT group_name [, ADD_EXTRACT_attribute] [, THREAD number] [, ETROLLOVER]  -- To change the attributes of an Extract group, To increment a trail to the next file in the sequence
ggsci> alter extract fin, begin 2012-02-16
ggsci> alter extract fin, etrollover
ggsci> alter extract fin, extseqno 26, extrba 338
ggsci> alter extract accounts, thread 4, begin 2012-03-09
ggsci> alter extract sales, lsn 1234:123:1
```
10. lag
```
ggsci> LAG EXTRACT group_name    -- To determine a true lag time between Extract and the datasource
ggsci> lag extract ext*
ggsci> lag extract *
```

## replicat相关
1. send rep2, NOHANDLECOLLISIONS
>send 命令动态取消HANDLECOLLISIONS


2. info replicat fin, showch
>复制进程的检查点信息

3. info replicat emp_rep, detail
>复制进程的相应信息

4. info checkpointtable gg_owner.chkpt_table
>检查点表信息

5. 常用添加replicat进程命令
```
ggsci> add replicat repl, exttrail C:\OGG10G\dirdat\lt
ggsci> add replicat repl, exttrail /oracle/gg11/dirdat/lt, checkpointtable gg_owner.checkpoint
ggsci> add replicat initload, specialrun
ggsci> add replicat repl, exttrail /oracle/gg11/dirdat/lt, nodbcheckpoint
```
6. stats
```
ggsci> STATS REPLICAT group_name [, statistic] [, TABLE table] [, TOTALSONLY table_specification] [, REPORTDETAIL | NOREPORTDETAIL] [, REPORTRATE HR|MIN|SEC] [, ... ]   -- To display statistics for one or more Replicat groups
ggsci> stats rep_hr
ggsci> stats replicat fin, total, table acct, reportrate hr, reset, noreportdetail
ggsci> STATS ER group_wildcard_specification   -- To get statistics on multiple Extract and Replicat groups as a unit
ggsci> stats er ext*
```
7. send
```
ggsci> SEND REPLICAT group_name,
{ FORCESTOP | GETLAG | HANDLECOLLISIONS [table_specification] | NOHANDLECOLLISIONS [table_specification] | REPORT [HANDLECOLLISIONS [table_specification]] | STATUS | STOP | TRACE[2] [DDLINCLUDE | DDL[ONLY]] [FILE] file_name | TRACE[2] OFF | TRACEINIT
}    -- To communicate with a starting or running Replicat process
ggsci> send replicat fin, handlecollisions
ggsci> send replicat fin, report handlecollisions fin_*
ggsci> send replicat fin, getlag
ggsci> SEND ER group_wildcard_specification   -- To send instructions to multiple Extract and Replicat groups as a unit
ggsci> send er *ext
```
8. lag
```
ggsci> LAG REPLICAT group_name   -- To determine a true lag time between Replicat and the trail
ggsci> lag replicat myrepl
ggsci> lag replicat *
```
## trail相关

1. INFO EXTTRAIL trail_name

2. info exttrail e:\ogg\dirdat\ex
>本地trail文件信息

3. INFO RMTTRAIL trail_name

4. info rmttrail d:\ogg\dirdat\ex
>远程trail文件信息

5. 常用添加trail 命令
```
ggsci> ADD EXTTRAIL trail_name, EXTRACT group_name [, MEGABYTES n] [, SEQNO n]   -- To create a trail for online processing on local system
ggsci> add exttrail /oracle/gg11/dirdat/lt, extract s_extr
ggsci> add exttrail C:\OGG10G\dirdat\et, extract emp_ext
ggsci> add exttrail c:\ogg\dirdat\fi, extract fin, megabytes 30
ggsci> ADD RMTTRAIL trail_name, EXTRACT group_name [, MEGABYTES n] [, SEQNO n]   -- To create a trail for online processing on remote system
ggsci> add rmttrail C:\OGG10G\dirdat\hr, extract extr
ggsci> add rmttrail /u01/app/oracle/ogg/dirdat/ms, extract msextr
ggsci> add rmttrail /u01/app/oracle/ogg/dirdat/my, extract mysql, megabytes 50
```
6. alter
```
ggsci> ALTER EXTTRAIL trail_name, EXTRACT group_name [, MEGABYTES n]   -- To change the attributes of a trail (on the local system)
ggsci> alter exttrail c:\ogg\dirdat\aa, extract fin, megabytes 30
ggsci> ALTER RMTTRAIL trail_name, EXTRACT group_name [, MEGABYTES n]   -- To change the attributes of a trail (on a remote system)
ggsci> alter rmttrail c:\ogg\dirdat\et, extract fin, megabytes 25
```
