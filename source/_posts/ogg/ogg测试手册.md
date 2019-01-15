---
title: OGG测试手册
date: 2019-01-03 00:00:00
tags: 
- ogg
categories: 
- tool
---
> ogg测试环境搭建
<!-- more -->
<!-- toc -->
# 源端数据库配置
```SQLEXEC
﻿sql> alter database archivelog;
sql> alter database add supplemental log data;
sql> create user ogg identified by ogg;
sql> grant connect,resource to ogg;
sql> grant execute on utl_file to ogg;
sql> grant select any dictionary,select any table to ogg;
sql> grant alter any table to ogg;
sql> grant flashback any table to ogg;
sql> grant execute on DBMS_FLASHBACK to ogg;
#直接赋dba权限即可，上面可以不用赋
sql> **grant dba to ogg;**
```


# 源端ogg配置
```
ogg> dblogin userid ogg password ogg
ogg>  add trandata smartecap_numone_xj.*
ogg>  create subdirs
ogg>  edit params mgr
PORT 7809
ogg> add trandata smartecap_numone_xj.*
ogg> info trandata smartecap_numone_xj.*
#也可以同步schema信息
ogg> add extract exts,tranlog ,begin now
ogg> info all
ogg> add exttrail ./dirdat/st,extract exts,megabytes 100
ogg> edit params exts

EXTRACT exts
SETENV (NLS_LANG = "AMERICAN_AMERICA.UTF8")
SETENV (ORACLE_HOME = "/home/oracle/app/oracle/product/11.2.0/dbhome_1")
USERID ogg, PASSWORD ogg
REPORTCOUNT EVERY 1 MINUTES, RATE
DISCARDFILE ./dirrpt/exts.dsc,APPEND,MEGABYTES 1024
DBOPTIONS  ALLOWUNUSEDCOLUMN
WARNLONGTRANS 2h,CHECKINTERVAL 350s
EXTTRAIL ./dirdat/st
FETCHOPTIONS NOUSESNAPSHOT
TRANLOGOPTIONS  CONVERTUCS2CLOBS
IGNOREDELETES
TABLE smartecap_numone_xj.*;

ogg>  add extract dpend,exttrailsource ./dirdat/st
ogg>  add rmttrail /home/oracle/app/oracle/ogg/dirdat/st ,extract dpend
ogg>  edit params dpend
EXTRACT dpend
SETENV (NLS_LANG = AMERICAN_AMERICA.UTF8)
USERID ogg, PASSWORD ogg
PASSTHRU
RMTHOST 192.168.106.213, MGRPORT 7809, compress
RMTTRAIL /home/oracle/app/oracle/ogg/dirdat/st
TABLE smartecap_numone_xj.*;

```

# EXTRACT进程参数配置说明：
1. SETENV：配置系统环境变量
2. USERID/ PASSWORD：指定OGG连接数据库的用户名和密码，这里使用3.4部分中创建的数据库用户OGG;
3. COMMENT：注释行，也可以用--来代替;
4. TABLE：定义需复制的表，后面需以;结尾
5. TABLEEXCLUDE：定义需要排除的表，如果在TABLE参数中使用了通配符，可以使用该参数指定排除掉得表。
6. GETUPDATEAFTERS|IGNOREUPDATEAFTERS：是否在队列中写入后影像，缺省复制
7. GETUPDATEBEFORES| IGNOREUPDATEBEFORES：是否在队列中写入前影像，缺省不复制
8. GETUPDATES|IGNOREUPDATES：是否复制UPDATE操作，缺省复制
9. GETDELETES|IGNOREDELETES：是否复制DELETE操作，缺省复制
10. GETINSERTS|IGNOREINSERTS：是否复制INSERT操作，缺省复制
11. GETTRUNCATES|IGNORETRUNDATES：是否复制TRUNCATE操作，缺省不复制；
12. RMTHOST：指定目标系统及其GoldengateManager进程的端口号，还用于定义是否使用压缩进行传输，本例中的compress为压缩传输；
13. RMTTRAIL：指定写入到目标断的哪个队列；
14. EXTTRAIL：指定写入到本地的哪个队列；
15. SQLEXEC：在extract进程运行时首先运行一个SQL语句；
16. PASSTHRU：禁止extract进程与数据库交互，适用于Data Pump传输进程；
17. REPORT：定义自动定时报告；
18. STATOPTIONS：定义每次使用stat时统计数字是否需要重置；
19. REPORTCOUNT：报告已经处理的记录条数统计数字；
20. TLTRACE：打开对于数据库日志的跟踪日志；
21. DISCARDFILE：定义discardfile文件位置，如果处理中油记录出错会写入到此文件中；
22. DBOPTIONS：指定对于某种特定数据库所需要的特殊参数；
23. TRANLOGOPTIONS：指定在解析数据库日志时所需要的特殊参数，例如：对于裸设备，可能需要加入以下参数 rawdeviceoggset 0
24. WARNLONGTRANS：指定对于超过一定时间的长交易可以在gsserr.log里面写入警告信息，本处配置为每隔3分钟检查一次场交易，对于超过2小时的进行警告；


# 目标端配置
```
ogg> create subdirs
ogg> dblogin userid ogg,password ogg
#需要checkpoint时 需指定。
ogg> add checkpointtable ogg.checkpoint
ogg> add replicat repnd,exttrail ./dirdat/st,checkpointtable ogg.checkpoint
ogg> edit param repnd
REPLICAT repnd
SETENV (NLS_LANG = "AMERICAN_AMERICA.UTF8")
SETENV (ORACLE_HOME = "/home/oracle/app/oracle/product/11.2.0/dbhome_1")
USERID ogg, PASSWORD ogg
HANDLECOLLISIONS
ASSUMETARGETDEFS
ddloptions report
MAP smartecap_numone_xj.*, TARGET smartecap_numone.*;
--DISCARDFILE /u01/app/oracle/ogg/discards.dsc, append, megabytes 1024
--TABLEEXCLUDE pdbjk.new_jk.SYS_EXPORT_SCHEMA*
```


# ogg用户最简配置示例
```
sql> create user tt identified by tt;
sql> grant connect,resource,dba to tt;
```
