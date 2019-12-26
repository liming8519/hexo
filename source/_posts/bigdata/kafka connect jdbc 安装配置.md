---
title: kafka connect jdbc 安装配置
tags: 
- kafka 
categories: 
- tool 
date: 2019-12-26 02:00:00
---

>  

<!-- more -->


## 安装
confluent官网下载kafka-connect-jdbc，获取kafka-connect-jdbc.jar包，把jar包放在kafka的lib下，同时下载mysql的驱动到kafka的lib目录下。

```
curl 192.168.106.119:8083/connector-plugins | python -m json.tool
[
    {
        "class": "io.confluent.connect.jdbc.JdbcSinkConnector",
        "type": "sink",
        "version": "5.3.2"
    },
    {
        "class": "io.confluent.connect.jdbc.JdbcSourceConnector",
        "type": "source",
        "version": "5.3.2"
    },
    {
        "class": "io.debezium.connector.mysql.MySqlConnector",
        "type": "source",
        "version": "1.0.0.Final"
    },
    {
        "class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
        "type": "sink",
        "version": "2.1.1"
    },
    {
        "class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
        "type": "source",
        "version": "2.1.1"
    }
]



```
## 关于kafka的scram的相关配置
```
[root@cloudsb kafka_2.12-2.1.1]# vi config/connect-distributed.properties 
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="car_record_user" password="123456";
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
```
## 配置
```
curl -s -X POST -H "Content-Type: application/json" --data \
'{
"name": "mysql-jdbc-connect",  
"config": {
"_comment": "kafka connect jdbc",
"connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector", 
"_comment": "数据库连接字符串",
"connection.url": "jdbc:mysql://192.168.106.248:3306/infoveriplatform_test?user=root&password=123456",
"_comment": "指定表类型",
"table.types": "table,view",
"_comment": "需要同步的表，如果想指定字段请使用视图过度",
"table.whitelist": "tb_control_car,tb_control_person,v_tb_car_record", 
"_comment": "根据时间增量获取数据",
"mode": "timestamp", 
"_comment": "时间列",
"timestamp.column.name": "create_time",
"_comment": "If the column is not defined as NOT NULL, tell the connector to ignore this  ",  
"validate.non.null": "false",
"_comment": "指定数据库，也可以在timestamp.column.name中指定dbname.tablename ",  
"catalog.pattern" : "infoveriplatform_test", 
"_comment": "拉取时间间隔",  
"poll.interval.ms" : "5000",
"_comment": " topic前缀",  
"topic.prefix": "hch-"
}
}' \
http://localhost:8083/connectors
```