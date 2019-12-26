---
title: kafka的SCRAM配置
tags: 
- kafka 
categories: 
- tool 
date: 2019-12-26 02:00:00
---

>  

<!-- more -->
## 创建SCRAM证书
> Kafka支持SCRAM-SHA-256和SCRAM-SHA-512，可与TLS一起使用执行安全认证。 用户名用作配置ACL等认证的Principal。Kafka中的默认SCRAM实现是在Zookeeper中存储SCRAM的证书，适用于Zookeeper在私有网络上的Kafka安装。

1. 为用户car_record_user创建SCRAM凭证（密码是123456）
```
bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[iterations=8192,password=123456],SCRAM-SHA-512=[password=123456]' --entity-type users --entity-name car_record_user
```
2. 为用户car_record_user创建SCRAM凭证（密码是123456）
```
bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=123456],SCRAM-SHA-512=[password=123456]' --entity-type users --entity-name admin
```


3. 可以使用--describe列出现有的证书
```
bin/kafka-configs.sh --zookeeper localhost:2181 --describe --entity-type users --entity-name car_record_user
```
4. 可以使用--delete为一个或多个SCRAM机制删除证书
```
bin/kafka-configs.sh --zookeeper localhost:2181 --alter --delete-config 'SCRAM-SHA-512' --entity-type users --entity-name car_record_user
```
## 配置kafka broker
1. 在每个Kafka broker的config目录下添加一个类似于下面的JAAS文件，我们姑且将其称为kafka_server_jaas.conf：
```
[root@cloudsb config]# pwd
/usr/local/kafka_2.12-2.1.1/config
[root@cloudsb config]# vi kafka_server_jaas.conf
KafkaServer {
 org.apache.kafka.common.security.scram.ScramLoginModule required
 username="admin"
 password="123456";
};
```
2. 其中，broker使用KafkaServer中的用户名和密码来和其他broker进行连接。 在这个例子中，admin是broker之间通信的用户。
```
[root@cloudsb bin]# pwd
/usr/local/kafka_2.12-2.1.1/bin
[root@cloudsb bin]# vi kafka-server-start.sh 
if [ "x$KAFKA_OPTS" = "x" ]; then
    export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/local/kafka_2.12-2.1.1/config/kafka_server_jaas.conf"
fi

```
3. 在server.properties中配置SASL端口和SASL机制。 
```
[root@managementa config]# pwd
/usr/local/kafka_2.12-2.1.1/config
[root@managementa config]# cp server.properties server_scram.properties
[root@managementa config]# vi server_scram.properties 
listeners=SASL_PLAINTEXT://:9092
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256## (or SCRAM-SHA-512)
sasl.enabled.mechanisms=SCRAM-SHA-256## (or SCRAM-SHA-512)

allow.everyone.if.no.acl.found=false
super.users=User:admin
authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
```
##  配置kafka客户端

为每个客户端配置JAAS配置（在producer.properteis或consumer.properteis）。登录模块展示了客户端（如生产者和消费者）如何连接到broker的。下面是配置了SCRAM机制的客户端的例子。
```
[root@managementa config]# vi kafka_client_jaas.conf
KafkaClient {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="car_record_user"
    password="123456";
};
```
客户端使用username和password来配置客户端连接的用户。在这个例子中，客户端使用用户car_record_user连接broker。在JVM中不同的客户端连接不同的用户（通过在sasl.jaas.config中指定不同的用户名和密码）。
客户端的JAAS配置通过指定作为JVM的参数。使用名为KafkaCLient的客户端登录。此选择仅允许来自JVM的所有客户端连接中的一个用户。
在 producer.properties 或 consumer.properties 中配置以下参数：
```
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256 ##(or SCRAM-SHA-512)
```
启动生产者消费者时先指定环境变量
```
export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/local/kafka_2.12-2.1.1/config/kafka_client_jaas.conf"
```
或者这样
```
[root@cloudsa kafka_2.12-2.1.1]# pwd
/usr/local/kafka_2.12-2.1.1
[root@cloudsa kafka_2.12-2.1.1]# cp bin/kafka-console-consumer.sh bin/kafka-console-consumer-scram.sh
[root@cloudsa kafka_2.12-2.1.1]# cp bin/kafka-console-producer.sh bin/kafka-console-producer-scram.sh
[root@cloudsa kafka_2.12-2.1.1]# vi bin/kafka-console-consumer-scram.sh
if [ "x$KAFKA_OPTS" = "x" ]; then
    export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/local/kafka_2.12-2.1.1/config/kafka_client_jaas.conf"
fi
[root@cloudsa kafka_2.12-2.1.1]# vi bin/kafka-console-producer-scram.sh 
if [ "x$KAFKA_OPTS" = "x" ]; then
    export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/local/kafka_2.12-2.1.1/config/kafka_client_jaas.conf"
fi
```
再指定参数
```
--consumer.config config/p_c.properties 
bin/kafka-console-consumer.sh --bootstrap-server localhost:9093 --topic test --consumer.config config/p_c.properties

```
## 启动与示例
```
[root@managementa kafka_2.12-2.1.1]# bin/kafka-server-start.sh -daemon config/server_scram.properties

[root@cloudsa kafka_2.12-2.1.1]# bin/kafka-console-consumer-scram.sh --bootstrap-server localhost:9092 --topic hch-v_tb_car_record --consumer.config config/p_c.properties  --from-beginning

```
## 另外一种客户端配置方式
```
[root@cloudsa kafka_2.12-2.1.1]# cat config/p_c.properties   
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
group.id=car-group
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="car_record_user" password="123456";
```
## acl配置示例
```
[root@cloudsa kafka_2.12-2.1.1]# bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal User:car_record_user --operation Read --topic hch-v_tb_car_record --group car-group


[root@cloudsa kafka_2.12-2.1.1]# bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=localhost:2181 --list


[root@cloudsa kafka_2.12-2.1.1]# vi config/p_c.properties 
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
group.id=car-group

[root@cloudsa kafka_2.12-2.1.1]# bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=localhost:2181 --remove --allow-principal User:car_record_user --operation Read --topic hch-v_tb_car_record --group car-group --force
```

## 查看用户

```
[root@cloudsb kafka_2.12-2.1.1]# bin/zookeeper-shell.sh localhost:2181
 ls /config/users
[admin, car_record_user]
```