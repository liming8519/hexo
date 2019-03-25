---
title: ogg for big data 12.3.2.1 using the elasticsearch (5.2.2) handler[再次]
date: 2019-03-25 00:00:00
tags:
- elasticsearch
- ogg
categories:
- tool
---
> elasticsearch 5.2.2是ogg for bigdata支持的最高版本
> elasticsearch 5.2.2 安装略
<!-- more -->

1. 源数据配置略，因与ogg普通配置一致。以下给出关键配置其他ogg相关的请参照ogg安装文档
```
dblogin userid ogg password ogg
add trandata smartecap_data.*
```
2. cat dirprm/elastic.prm

```
REPLICAT elastic
TARGETDB LIBFILE libggjava.so SET property=./dirprm/elastic.properties 
MAP smartecap_data.*, TARGET smartecap_data.*;
```

3. cat dirprm/elastic.properties 
```
#any name of your choice
gg.handlerlist=elasticsearch
#type of handler to use ex Elasticsearch , kafka and so on
gg.handler.elasticsearch.type=elasticsearch

## Handler properties for Elasticsearch 5.x, 逗号分隔
gg.handler.elasticsearch.ServerAddressList=192.168.106.213:9300
gg.handler.elasticsearch.clientSettingsFile=client5.properties
gg.handler.elasticsearch.version=5.x
#gg.handler.elasticsearch.bulkWrite=true
gg.handler.elasticsearch.upsert=true
#Set the Classpath , Default location of 5.X JARs, classpath 中一定要有elasticsearch的客户端驱动。
gg.classpath=./dirprm*:/home/es/elasticsearch-5.2.2/lib/*:/home/es/elasticsearch-5.2.2/modules/transport-netty3/*:/home/es/elasticsearch-5.2.2/modules/transport-netty4/*:/home/es/elasticsearch-5.2.2/modules/reindex/*


#goldengate.userexit.writers=javawriter
#javawriter.stats.display=TRUE
#javawriter.stats.full=TRUE
gg.log=log4j
gg.log.level=INFO
gg.report.time=30sec
#javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=.:ggjava/ggjava.jar:./dirprm
```
4. cat dirprm/client5.properties 
```
#集群名称
cluster.name=oggtest
#xpack.security.user=<x-pack user name>:<x-pack password>
```

5. Elasticsearch 5.1.2 with X-Pack 5.1.2
```
commons-codec-1.10.jar
commons-logging-1.1.3.jar
compiler-0.9.3.jar
elasticsearch-5.1.2.jar
HdrHistogram-2.1.6.jar
hppc-0.7.1.jar
httpasyncclient-4.1.2.jar
httpclient-4.5.2.jar
httpcore-4.4.5.jar
httpcore-nio-4.4.5.jar
jackson-core-2.8.1.jar
jackson-dataformat-cbor-2.8.1.jar
jackson-dataformat-smile-2.8.1.jar
jackson-dataformat-yaml-2.8.1.jar
jna-4.2.2.jar
joda-time-2.9.5.jar
jopt-simple-5.0.2.jar
lang-mustache-client-5.1.2.jar
lucene-analyzers-common-6.3.0.jar
lucene-backward-codecs-6.3.0.jar
lucene-core-6.3.0.jar
lucene-grouping-6.3.0.jar
lucene-highlighter-6.3.0.jar
lucene-join-6.3.0.jar
lucene-memory-6.3.0.jar
lucene-misc-6.3.0.jar
lucene-queries-6.3.0.jar
lucene-queryparser-6.3.0.jar
lucene-sandbox-6.3.0.jar
lucene-spatial3d-6.3.0.jar
lucene-spatial-6.3.0.jar
lucene-spatial-extras-6.3.0.jar
lucene-suggest-6.3.0.jar
netty-3.10.6.Final.jar
netty-buffer-4.1.6.Final.jar
netty-codec-4.1.6.Final.jar
netty-codec-http-4.1.6.Final.jar
netty-common-4.1.6.Final.jar
netty-handler-4.1.6.Final.jar
netty-resolver-4.1.6.Final.jar
netty-transport-4.1.6.Final.jar
percolator-client-5.1.2.jar
reindex-client-5.1.2.jar
rest-5.1.2.jar
securesm-1.1.jar
snakeyaml-1.15.jar
t-digest-3.0.jar
transport-5.1.2.jar
transport-netty3-client-5.1.2.jar
transport-netty4-client-5.1.2.jar
x-pack-transport-5.1.2.jar
```
6. 其中dirprm中需要的esearch驱动有
```
lang-mustache-client-5.2.2.jar
percolator-client-5.2.2.jar
transport-5.2.2.jar
x-pack-transport-5.2.2.jar
```