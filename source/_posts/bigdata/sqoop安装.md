---
title: sqoop安装
tags:
  - sqoop
categories:
  - db
date: 2019-04-011 02:00:00

---


> sqoop安装
<!-- more -->
## 安装
1. 解压即可
```
[hadoop@hadoop3 ~]$ tar -zxvf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/sqoop147
[hadoop@hadoop3 apps]$ cd /opt/sqoop147/sqoop-1.4.7/conf
[hadoop@hadoop3 conf]$ mv sqoop-env-template.sh sqoop-env.sh
```
## 使用
1. 加入mysql驱动到sqoop的lib目录
2. 环境变量
```
#Sqoop
export SQOOP_HOME=/usr/local/sqoop147
export PATH=$PATH:$SQOOP_HOME/bin

export HADOOP_COMMON_HOME=/opt/hadoop285/hadoop-2.8.5
export HADOOP_MAPRED_HOME=/opt/hadoop285/hadoop-2.8.5
```
3. 测试
```
sqoop version
sqoop help
sqoop list-databases --connect jdbc:mysql://192.168.11.99:3306/mysql?characterEncoding=UTF-8 --username root --password '123456'
sqoop list-tables --connect jdbc:mysql://192.168.11.99:3306/xzinside_dev?characterEncoding=UTF-8 --username root --password '123456' 
```
4. 配置hbase
```
export HBASE_HOME=/opt/hbase214/hbase-2.1.4
export HBASE_CLASSPATH=/opt/hbase214/hbase-2.1.4/conf
```