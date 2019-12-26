---
title: kafka的SASL配置
tags: 
- kafka 
categories: 
- tool 
date: 2019-05-27 02:00:00
---
> kafka的SASL配置
<!-- more -->


## server.properties的配置
> 可以重新制定一个配置文件如server_sasl.properties
> 到kafka的config文件加下
```
listeners=SASL_PLAINTEXT://xx.xx.xx.xx:8123
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.enabled.mechanisms=PLAIN
sasl.mechanism.inter.broker.protocol=PLAIN


touch kafka_server_jaas.conf
vi kafka_server_jaas.conf
KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="kafka"#这个帐号是下面配置的
    password="password"#这个密码是下面配置的
    ##解释，下面的格式是user_用户名="密码"，而上面broker使用的帐号密码就是下面配置的，上面broker使用的帐号密码分别是kafka和password，而kafka和password是下面设置的。
    user_kafka="password"
    user_mooc="mooc";
};

touch kafka_client_jaas.conf
vi kafka_client_jaas.conf

KafkaClient {
        org.apache.kafka.common.security.plain.PlainLoginModule required
        username="mooc"
        password="mooc";
};
```
## 命令文件的配置
```
cd ../bin
vi kafka-server-start.sh
if [ "x$KAFKA_OPTS" = "x" ]; then
    export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/local/kafka_2.12-2.1.1/config/kafka_server_jaas.conf"
fi


vi kafka-console-producer.sh
vi kafka-console-consumer.sh
if [ "x$KAFKA_OPTS" = "x" ]; then
    export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/local/kafka_2.12-2.1.1/config/kafka_client_jaas.conf"
fi
```
## 启动kafka
```
./kafka-server-start.sh -daemon ../config/server.properties 
```
## 其他相关脚本
```
./kafka-topics.sh  --zookeeper localhost:2181 --list

./kafka-topics.sh  --zookeeper localhost:2181 --create --topic Test  --partitions 1 --replication-factor 1

./kafka-console-producer.sh --broker-list 192.168.11.99:9092 --topic test --producer-property security.protocol=SASL_PLAINTEXT --producer-property sasl.mechanism=PLAIN

./kafka-console-consumer.sh --bootstrap-server 192.168.11.99:9092 --from-beginning --topic test --consumer-property security.protocol=SASL_PLAINTEXT --consumer-property sasl.mechanism=PLAIN
```