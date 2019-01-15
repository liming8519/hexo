---
title: ogg for bigdata + elasticsearch 5.2.2
date: 2019-01-02 00:00:00
tags:
- elasticsearch
- ogg
categories:
- tool
---

> elasticsearch 5.2.2是ogg for bigdata支持的最高版本
> elasticsearch 5.2.2 安装略
<!-- more -->

1. 源数据库配置

```
ogg> add trandata smartecap_data.*
ogg> add extract ext_data,tranlog ,begin now
ogg> add exttrail ./dirdat/st,extract ext_data,megabytes 100
ogg> edit params ext_data
EXTRACT ext_data
SETENV (NLS_LANG = "AMERICAN_AMERICA.ZHS16GBK")
SETENV (ORACLE_HOME = "/home/oracle/app/oracle/product/11.2.0/dbhome_1")
USERID ogg, PASSWORD ogg
REPORTCOUNT EVERY 1 MINUTES, RATE
--DISCARDFILE ./dirrpt/ext_data.dsc,APPEND,MEGABYTES 1024
DBOPTIONS  ALLOWUNUSEDCOLUMN
WARNLONGTRANS 2h,CHECKINTERVAL 350s
EXTTRAIL ./dirdat/dt
FETCHOPTIONS NOUSESNAPSHOT
--TRANLOGOPTIONS CONVERTUCS2CLOBS
--IGNOREDELETES
TABLE smartecap_data.dt*;
ogg> start ext_data
ogg> add extract PUMP_DT,exttrailsource ./dirdat/dt
ogg> add rmttrail ./dirdat/dt ,extract PUMP_DT
ogg> edit params PUMP_DT
EXTRACT pump_dt
SETENV (NLS_LANG = AMERICAN_AMERICA.UTF8)
USERID ogg, PASSWORD ogg
PASSTHRU
RMTHOST 192.168.106.213, MGRPORT 7801, compress
RMTTRAIL ./dirdat/dt
TABLE smartecap_data.*;
ogg> start pump_dt

```

2. 目标ogg for bigdata配置

```
$ cat elastic.properties
gg.handlerlist=elasticsearch
gg.handler.elasticsearch.type=elasticsearch
## Handler properties for Elasticsearch 6.x
gg.handler.elasticsearch.ServerAddressList=192.168.106.213:9300
gg.handler.elasticsearch.clientSettingsFile=client6.properties
gg.handler.elasticsearch.version=5.x
#gg.handler.elasticsearch.bulkWrite=true
gg.classpath=./dirprm*:/home/es/elasticsearch-5.2.2/lib/*:/home/es/elasticsearch-5.2.2/modules/transport-netty3/*:/home/es/elasticsearch-5.2.2/modules/transport-netty4/*:/home/es/elasticsearch-5.2.2/modules/reindex/*
goldengate.userexit.writers=javawriter
javawriter.stats.display=TRUE
javawriter.stats.full=TRUE
gg.log=log4j
gg.log.level=INFO
gg.report.time=30sec
javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=.:ggjava/ggjava.jar:./dirprm
$ cat client6.properties
cluster.name=oggtest
#xpack.security.user=<x-pack user name>:<x-pack password>

ogg> add replicat ELASTIC,exttrail ./dirdat/dt
ogg> edit param ELASTIC
REPLICAT elastic
TARGETDB LIBFILE libggjava.so SET property=./dirprm/elastic.properties
MAP smartecap_data.*, TARGET smartecap_data.*;
#java的配置，java需要jdk1.8
$ cat .bash_profile
export JAVA_HOME=/usr/local/jdk
PATH=$PATH:$HOME/bin:/usr/local/jdk/bin
export PATH
export LD_LIBRARY_PATH=/usr/local/jdk/jre/lib/amd64/server:$LD_LIBRARY_PATH
#elasticsearch 客户端jar 需要下载放到dirprm中，需要下载的jar如下
$ ls *.jar
lang-mustache-client-5.2.2.jar  percolator-client-5.2.2.jar  transport-5.2.2.jar  x-pack-transport-5.2.2.jar

ogg> start ELASTIC
```

>jar文件下载

jar文件名  | 下载  
--|---
lang-mustache-client-5.2.2.jar  | [点](https://raw.githubusercontent.com/zixujing/book1/master/code/lang-mustache-client-5.2.2.jar)
percolator-client-5.2.2.jar  | [点](https://raw.githubusercontent.com/zixujing/book1/master/code/percolator-client-5.2.2.jar)
transport-5.2.2.jar  |  [点](https://raw.githubusercontent.com/zixujing/book1/master/code/transport-5.2.2.jar)   
x-pack-transport-5.2.2.jar  |   [点](https://raw.githubusercontent.com/zixujing/book1/master/code/x-pack-transport-5.2.2.jar)
