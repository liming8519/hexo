---
title: kafka-eagle安装配置
tags: 
- kafka 
categories: 
- tool 
date: 2019-12-27 02:00:00
---

>  

<!-- more -->

## 安装

```
源：http://download.kafka-eagle.org/
[root@cloudsa ~]# wget https://codeload.github.com/smartloli/kafka-eagle-bin/tar.gz/v1.4.2
[root@cloudsa ~]# mv kafka-eagle-web-1.4.2/ /usr/local/
[root@cloudsa ~]# cd /usr/local/kafka-eagle-web-1.4.2/
```

## 调整配置

```
[root@cloudsa kafka-eagle-web-1.4.2]# vi conf/system-config.properties 
######################################
# multi zookeeper&kafka cluster list
######################################
kafka.eagle.zk.cluster.alias=test-kafka
test-kafka.zk.list=192.168.106.117:2181,192.168.106.118:2181,192.168.106.119:2181
#cluster2.zk.list=xdn10:2181,xdn11:2181,xdn12:2181

######################################
# zk client thread limit
######################################
kafka.zk.limit.size=25

######################################
# kafka eagle webui port
######################################
kafka.eagle.webui.port=8048

######################################
# kafka offset storage
######################################
test-kafka.kafka.eagle.offset.storage=kafka
#cluster2.kafka.eagle.offset.storage=zk

######################################
# kafka metrics, 30 days by default
######################################
kafka.eagle.metrics.charts=false
kafka.eagle.metrics.retain=30


######################################
# kafka sql topic records max
######################################
kafka.eagle.sql.topic.records.max=5000
kafka.eagle.sql.fix.error=false

######################################
# delete kafka topic token
######################################
kafka.eagle.topic.token=keadmin

######################################
# kafka sasl authenticate
######################################
test-kafka.kafka.eagle.sasl.enable=true
test-kafka.kafka.eagle.sasl.protocol=SASL_PLAINTEXT
test-kafka.kafka.eagle.sasl.mechanism=SCRAM-SHA-256
test-kafka.kafka.eagle.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="admin" password="123456";
test-kafka.kafka.eagle.sasl.client.id=

#cluster2.kafka.eagle.sasl.enable=false
#cluster2.kafka.eagle.sasl.protocol=SASL_PLAINTEXT
#cluster2.kafka.eagle.sasl.mechanism=PLAIN
#cluster2.kafka.eagle.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="kafka" password="kafka-eagle";
#cluster2.kafka.eagle.sasl.client.id=

######################################
# kafka sqlite jdbc driver address
######################################
#kafka.eagle.driver=org.sqlite.JDBC
#kafka.eagle.url=jdbc:sqlite:/hadoop/kafka-eagle/db/ke.db
#kafka.eagle.username=root
#kafka.eagle.password=www.kafka-eagle.org

######################################
# kafka mysql jdbc driver address
######################################
kafka.eagle.driver=com.mysql.jdbc.Driver
kafka.eagle.url=jdbc:mysql://192.168.106.248:3306/ke?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
kafka.eagle.username=root
kafka.eagle.password=123456

[root@cloudsa kafka-eagle-web-1.4.2]# vi bin/ke.sh 
export JAVA_HOME=/opt/jdk1.8.0_144
export KE_HOME=/usr/local/kafka-eagle-web-1.4.2
export PATH=$PATH:$KE_HOME/bin:$JAVA_HOME/bin

mysql> create database ke;

```
## 启动
```
[root@cloudsa kafka-eagle-web-1.4.2]# bin/ke.sh start

Version 1.4.2
*******************************************************************
* Kafka Eagle Service has started success.
* Welcome, Now you can visit 'http://192.168.106.118:8048/ke'
* Account:admin ,Password:123456
*******************************************************************
* <Usage> ke.sh [start|status|stop|restart|stats] </Usage>
* <Usage> https://www.kafka-eagle.org/ </Usage>
```