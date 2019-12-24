---
title: debezium+kafka 解析mysql的binlog日志
tags: 
- kafka 
categories: 
- tool 
date: 2019-12-23 02:00:00
---
> 

<!-- more -->

## mysql环境准备
### 开启binlog
```
[mysqld]
log-bin=mariadb-bin
binlog-format=ROW
server_id=100
```
### 创建账号

```
MariaDB [(none)]> create user 'deb'@'%' identified by 'deb'; 
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant reload,show databases,select ,replication slave,replication client on *.* to 'deb'@'%';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
mysql> create database deb;
mysql> use deb
mysql> create table tb_test (a varchar(12),b int ,c varchar(24));
```
## debezium
### 下载debezium
安装在119
```
wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/1.0.0.Final/debezium-connector-mysql-1.0.0.Final-plugin.tar.gz
```
### 安装debezium
安装在119
```
[root@cloudsc ~]# tar -xzvf debezium-connector-mysql-1.0.0.Final-plugin.tar.gz 
[root@cloudsc ~]# mv debezium-connector-mysql /usr/local/ 

##[root@cloudsc ~]# vi .bash_profile 
##export CLASSPATH=/usr/local/debezium-connector-mysql/debezium-core-1.0.0.Final.jar:.
##[root@cloudsc ~]# source .bash_profile 

```
## connect

### 配置文件并启动
登录119执行
```
[root@cloudsc ~]# cd /usr/local/debezium-connector-mysql/
[root@cloudsc debezium-connector-mysql]# ls
antlr4-runtime-4.7.2.jar  CONTRIBUTE.md  debezium-connector-mysql-1.0.0.Final.jar  debezium-ddl-parser-1.0.0.Final.jar  LICENSE.txt                             mysql-connector-java-8.0.16.jar
CHANGELOG.md              COPYRIGHT.txt  debezium-core-1.0.0.Final.jar             LICENSE-3rd-PARTIES.txt              mysql-binlog-connector-java-0.19.1.jar  README.md
##[root@cloudsc debezium-connector-mysql]# vi mysql.prop
{
  "name": "mariadb-connector", 
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector", 
    "tasks.max": "1",
    "database.hostname": "192.168.106.128", 
    "database.port": "3306", 
    "database.user": "deb", 
    "database.password": "deb", 
    "database.server.id": "184054", 
    "database.server.name": "mariadbdata", 
    "database.whitelist": "deb", 
    "database.history.kafka.bootstrap.servers": "192.168.106.117:9092,192.168.106.118:9092,192.168.106.119:9092", 
    "database.history.kafka.topic": "dbhistory.deb", 
    "column.blacklist": "deb.tb_test.b",
    "include.schema.changes": "false" 
  }
}
[root@cloudsb debezium-connector-mysql]# cd ../kafka_2.12-2.1.1/
[root@cloudsb debezium-connector-mysql]# cp mysql-connector-java-8.0.16.jar ../kafka_2.12-2.1.1/libs/
[root@cloudsb debezium-connector-mysql]# cp *.jar ../kafka_2.12-2.1.1/libs/
##[root@cloudsb kafka_2.12-2.1.1]# vi config/connect-standalone.properties 
##plugin.path=/usr/local/debezium-connector-mysql
##[root@cloudsb kafka_2.12-2.1.1]# bin/connect-standalone.sh config/connect-standalone.properties ../debezium-connector-mysql/mysql.prop 报错
[root@cloudsb kafka_2.12-2.1.1]#  bin/connect-distributed.sh config/connect-distributed.properties 
[root@cloudsb kafka_2.12-2.1.1]# curl -s -X POST -H "Content-Type: application/json" --data \
'{
  "name": "mariadb-connector", 
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector", 
    "tasks.max": "1",
    "database.hostname": "192.168.106.128", 
    "database.port": "3306", 
    "database.user": "deb", 
    "database.password": "deb", 
    "database.server.id": "184054", 
    "database.server.name": "mariadbdata", 
    "database.whitelist": "deb", 
    "database.history.kafka.bootstrap.servers": "192.168.106.117:9092,192.168.106.118:9092,192.168.106.119:9092", 
    "database.history.kafka.topic": "dbhistory.deb", 
    "column.blacklist": "deb.tb_test.b",
    "include.schema.changes": "false" 
  }
}' \
http://localhost:8083/connectors
```
## 使用

```
[root@cloudsb debezium-connector-mysql]# curl localhost:8083/connector-plugins
[root@cloudsb debezium-connector-mysql]# curl localhost:8083/connectors/mariadb-connector
[root@cloudsb debezium-connector-mysql]# curl localhost:8083/connectors                                         
["mariadb-connector"][root@cloudsb debezium-connector-mysql]# 

GET /Connectors：返回活跃的 Connector 列表
POST /Connectors：创建一个新的 Connector；请求的主体是一个包含字符串name字段和对象 config 字段（Connector 的配置参数）的 JSON 对象。
GET /Connectors/{name}：获取指定 Connector 的信息
GET /Connectors/{name}/config：获取指定 Connector 的配置参数
PUT /Connectors/{name}/config：更新指定 Connector 的配置参数
GET /Connectors/{name}/status：获取 Connector 的当前状态，包括它是否正在运行，失败，暂停等。
GET /Connectors/{name}/tasks：获取当前正在运行的 Connector 的任务列表。
GET /Connectors/{name}/tasks/{taskid}/status：获取任务的当前状态，包括是否是运行中的，失败的，暂停的等，
PUT /Connectors/{name}/pause：暂停连接器和它的任务，停止消息处理，直到 Connector 恢复。
PUT /Connectors/{name}/resume：恢复暂停的 Connector（如果 Connector 没有暂停，则什么都不做）
POST /Connectors/{name}/restart：重启 Connector（Connector 已故障）
POST /Connectors/{name}/tasks/{taskId}/restart：重启单个任务 (通常这个任务已失败)

DELETE /Connectors/{name}：删除 Connector, 停止所有的任务并删除其配置
```
