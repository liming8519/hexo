---
title: kafka手册
date: 2019-01-03 00:00:00
tags:
- kafka
- ogg
categories:
- tool
---

> 以下以2.0.0为例，0.11.0.3类似
<!-- more -->
## kafka安装
```bash
$ tar -xvzf kafka_2.11-2.0.0.tar.gz -C /usr/local/elk/
$ cd /usr/local/elk/kafka_2.11-2.0.0/config
$ vim zookeeper.properties
# zookeeper的data路径
dataDir=/usr/local/elk/kafka_2.11-2.0.0/zookeeper/data
# zookeeper的log路径
dataLogDir=/usr/local/elk/kafka_2.11-2.0.0/zookeeper/logs
# zookeeper的端口号
clientPort=2181
# zookeeper的客户端连接数，设置成0为无限制连接。
maxClientCnxns=0
$ #配置server.properties文件，修改该配置文件中的以下参数：
$ vim server.properties
#服务端口，默认9092
port=9092
#监听地址
host.name=192.168.106.xxx
#日志文件存储路径
log.dirs=/usr/local/elk/kafka_2.11-2.0.0/kafka/logs
#Zookeeper quorum设置，如果有多个使用逗号分割。
zookeeper.connect=192.168.106.226:2181
$ #启动zookeeper
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/zookeeper-server-start.sh  config/zookeeper.properties 1>/dev/null 2>&1 &
$ #启动kafka
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/kafka-server-start.sh  config/server.properties 1>/dev/null 2>&1 &
$ #停止zookeeper
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/zookeeper-server-stop.sh  config/zookeeper.properties
$ #停止kafka
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/kafka-server-stop.sh  config/server.properties
```
## kafka测试
```bash
$ #执行下面的命令创建一个topic，名称为“testTopic”：
$ cd /usr/local/elk/kafka_2.11-2.0.0
$ bin/kafka-topics.sh --create --zookeeper 192.168.0.xxx:2181 --replication-factor 1 --partitions 1 --topic testTopic
$ #执行下面的命令查看topic：
$ bin/kafka-topics.sh --list --zookeeper 192.168.0.xxx:2181
$ #执行下面的命令测试生产消息
$ bin/kafka-console-producer.sh --broker-list 192.168.0.xxx:9092 --topic testTopic
this is test
$ #执行下面的命令测试消费消息
$ bin/kafka-console-consumer.sh --bootstrap-server 192.168.0.xxx:9092 --topic testTopic --from-beginning
```
# broker配置
* broker.id: 默认值是0，也可以被设置成其他任意整数。这个值在整个kafka集群里必须是唯一的。
* port：默认9092
* zookeeper.connect:指定zookeeper地址
* log.dirs：kafka把所有的消息都保存在磁盘上，存放这些日志片段的目录是通过log.dirs指定的。它是一组用逗号分隔的本地文件系统路径。
* num.recovery.threads.per.data.dir:如果num.recovery.threads.per.data.dir设置8，并且log.dir指定了3个路径，那么总共需要24个线程。
* auto.create.topics.enable:建议设置为false，因为很多情况是非预期的
# 主题的默认配置
* num.partitions：指定了新创建的主题将包含多少个分区。
* log.retention.ms：根据时间决定数据可以被保留多久。
* log.retention.bytes：保留消息字节数来判断消息是否过期。如果log.retention.bytes被设置为1GB，主题有8个分区。那么最多可以保留8GB数据。
* log.segment.bytes:当日志片段大小达到log.segment.bytes指定的上限（默认是1GB）时，当前日志片段就会被关闭，一个新的日志片段被打开。如果一个主题每天只接受100MB的消息，而log.segment.bytes使用的默认设置，那么需要10天时间才能填满一个日志片段。因为日志片段关闭之前消息是不会过期的，所以如果log.retention.ms被设置为604800000（也就是1周），那么日志片段最多需要17天才会过期。
* log.segment.ms:默认没值。与log.segment.bytes哪个首先满足哪个先触发。
* message.max.bytes:限制单个消息的大小，默认是1M

# linux系统配置
## 虚拟内存
* vm.swappiness：值0在高版本的linux中明确表示，永远不产生交换。最合适的值是1.
* vm.dirty_background_ratio：设置小于10，一般设置为5.该值指的是系统内存的百分比。设置系统何时刷脏页到磁盘。
* vm.dirty_ratio：可以增加被内核进程刷新到磁盘之前的脏页数量，可以将它设置为大于20的值。60~80比较合理。
* 脏页数量如何查看。cat /proc/vmstat | egrep "dirty|writeback"
nr_dirty 3875
nr_writeback 29
nr_writeback_temp 0
## 网络
* net.core.wmen_default,net.core.rmem_default:合理的值是131072（128k）
* net.core.wmen_max,net.core.rmen_max：合理的值是2097152（2M）
* net.ipv4.tcp_wmen,net.ipv4.tcp_rmen:tcp socket的读写缓冲区，由3个整数组成，用空格分隔，分别表示最小值、默认值和最大值。最大值不能超过net.core.wmen_max和net.core.rmen_max.例如4096 65536 2048000
* net.ipv4.tcp_window_scaling:1
* net.ipv4.tcp_max_syn_backlog:大于1024
* net.core.netdev_max_baxklog:大于1000


















***
