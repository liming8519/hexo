---
title: maxwell+bireme+kafka+greenplum  mysql增量数据抽取到greenplum
tags:
  - maxwell bireme
categories:
  - db
date: 2019-05-29 02:00:00

---


> maxwell+bireme+kafka+greenplum  mysql增量数据抽取到greenplum
<!-- more -->
## Maxwell 数据源
### 配置mysql
```
vi /etc/my.cnf
[mysqld]
server-id=1
log-bin=master
binlog_format=row

mysql -uroot -pxxxx

mysql> create user 'maxwell'@'%' identified by 'maxwell';
mysql> GRANT ALL on maxwell.* to 'maxwell'@'%';
mysql> GRANT SELECT, REPLICATION CLIENT, REPLICATION SLAVE on *.* to 'maxwell'@'%';
如果是mysql8 则执行alter user 'maxwell'@'%' identified with mysql_native_password by 'maxwell';
mysql> CREATE DATABASE test;
mysql> create table test.test (id int primary key, name varchar(10));
```

### 创建kafka的topic
```
bin/kafka-topics.sh --create --topic maxwell --zookeeper 192.168.11.97:2181 --partitions 3 --replication-factor 2
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

### 在GreenPlum中新建表
```
create database maxwell;
\c maxwell;
create table test(id int primary key,name varchar(10));
```
### 启动maxwell
```
bin/maxwell --daemon --user='maxwell' --password='maxwell' --host=192.168.11.99 --include_dbs=test --include_tables=test --producer=kafka --kafka_topic=maxwell --kafka.bootstrap.servers=192.168.11.99:9092


bin/maxwell --user='maxwell' --password='maxwell' --host=192.168.11.99 --producer=stdout
查看一下
bin/kafka-console-consumer.sh --topic maxwell --from-beginning --bootstrap-server 192.168.11.99:9092
```
### maxwell配置文件配置

```
vi config
user=maxwell
password=maxwell
host=192.168.11.113
include_dbs=infoveriplatform_test
include_tables=tb_car_record,tb_control_car,tb_control_person,tb_person_record,tb_key_car_record,tb_key_person_record,tb_security_clearance
producer=kafka
kafka_topic=maxwell
kafka.bootstrap.servers=192.168.11.97:9092,192.168.11.98:9092,192.168.11.99:9092
kafka_partition_hash=murmur3
producer_partition_by=primary_key

./maxwell --daemon --config=config
```

### 关于XA问题解决
 I don't know whether to add this code in com.zendesk.maxwell.schema.ddl.SchemaChange.java is best way to solve this problem,but can start maxwell normally.
```
SQL_BLACKLIST.add(Pattern.compile("\\s*XA\\s+", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE));
```
## bireme 配置及启动
### 配置
```
cd 
vi `pwd`/etc/config.properties
target.url = jdbc:postgresql://192.168.11.84:5432/maxwell
target.user = gpadmin
target.passwd = gpadmin

data_source = mysql
mysql.type = maxwell
mysql.kafka.server = 192.168.11.99:9092
mysql.kafka.topic = maxwell
state.server.addr = 192.168.11.111

 vi `pwd`/etc/mysql.properties
test.test=public.test
```
###启动

```
需要关联软件apache-commons-daemon-jsvc-1.0.13-7.el7.x86_64
bin/bireme start
```
###结束
```
bin/bireme stop
```
