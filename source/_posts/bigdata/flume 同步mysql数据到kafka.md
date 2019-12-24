---
title: flume 同步mysql数据到kafka
date: 2019-12-24 00:00:00
tags:
- kafka
categories:
- tool
---
> 

<!-- more -->
## flume 同步mysql数据到kafka
### 资源
```
http://www.apache.org/dyn/closer.lua/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
https://github.com/keedio/flume-ng-sql-source/archive/v1.5.2.tar.gz

```
### flume安装
```
[root@cloudsb ~]# cd /opt
[root@cloudsb opt]# tar -xzvf apache-flume-1.9.0-bin.tar.gz 
[root@cloudsb opt]# cd apache-flume-1.9.0-bin
[root@cloudsb apache-flume-1.9.0-bin]# bin/flume-ng version
Flume 1.9.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: d4fcab4f501d41597bc616921329a4339f73585e
Compiled by fszabo on Mon Dec 17 20:45:25 CET 2018
From source with checksum 35db629a3bda49d23e9b3690c80737f9
[root@cloudsb apache-flume-1.9.0-bin]# cd lib
把flume ng sql 和mysql驱动jar包放在这个目录下，文件名字是mysql-connector-java-5.1.46.jar和flume-ng-sql-source-1.5.2.jar


```

### 配置mysql source
```
[root@cloudsb apache-flume-1.9.0-bin]# vi conf/mysql_control_car.conf
########## name ###################
a1.channels = ch-1
a1.sources = src-1
a1.sinks = k1
###########sql source#################
# For each one of the sources, the type is defined
a1.sources.src-1.type = org.keedio.flume.source.SQLSource
# mysql地址
a1.sources.src-1.hibernate.connection.url = jdbc:mysql://192.168.106.248:3306/infoveriplatform_test
# Hibernate Database connection properties
#数据库账号
a1.sources.src-1.hibernate.connection.user = root
#数据库密码
a1.sources.src-1.hibernate.connection.password = 123456
#是否自动提交
a1.sources.src-1.hibernate.connection.autocommit = true
a1.sources.src-1.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
a1.sources.src-1.hibernate.connection.driver_class = com.mysql.jdbc.Driver
#查询间隔
a1.sources.src-1.run.query.delay=100000
#输出路径
a1.sources.src-1.status.file.path = /root
#输出文件名称
a1.sources.src-1.status.file.name = sqlSource.status
# Custom query
#从哪里开始读取数据传输
a1.sources.src-1.start.from = 0
#SQL--传什么写什么
a1.sources.src-1.custom.query = SELECT cphm,hpzl,czsfzh,czxm from  tb_control_car
#批量发送数据量 应该是source 发送到 channel 
a1.sources.src-1.batch.size = 1000
#最大查询行数
a1.sources.src-1.max.rows = 100000
a1.sources.src-1.hibernate.connection.provider_class = org.hibernate.connection.C3P0ConnectionProvider
a1.sources.src-1.hibernate.c3p0.min_size=1
a1.sources.src-1.hibernate.c3p0.max_size=10
#分割符
a1.sources.sqlSource.delimiter.entry = |


################################################################
a1.channels.ch-1.type = memory
a1.channels.ch-1.capacity = 1000000
a1.channels.ch-1.transactionCapacity = 1000000
a1.channels.ch-1.byteCapacityBufferPercentage = 20
#a1.channels.ch-1.byteCapacity = 1000000

################################################################
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
#要传输的topic
a1.sinks.k1.topic = control_car
#broker地址
a1.sinks.k1.brokerList = 192.168.106.119:9092
#ack模式选择 -1.0,1
a1.sinks.k1.requiredAcks = 1
#批量发送数据量 应该是sink发送到 kafka 
a1.sinks.k1.batchSize = 200
a1.sinks.k1.channel = c1

a1.sinks.k1.channel = ch-1
a1.sources.src-1.channels=ch-1

```
### 运行flume
```
[root@cloudsb apache-flume-1.9.0-bin]# bin/flume-ng agent -n a1 -c conf -f conf/mysql_control_car.conf -Dflume.root.logger=INFO,console
```

### 总结

```
flume 只能根据自增id（number）来增量。
```