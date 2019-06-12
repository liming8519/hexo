---
title: kafka 集群安装
tags: 
- kafka 
categories: 
- tool 
date: 2019-05-07 02:00:00
---
> kafka 2.12-2.1.1 集群安装
<!-- more -->

## jdk安装
```
$ cd /opt
$ tar -xzvf jdk-8u161-linux-x64.tar.gz
$ vi ~/.bash_profile
JAVA_HOME=/opt/jdk1.8.0_161
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export PATH
$ source ~/.bash_profile
```

## zk安装

```
$ tar -xzvf kafka_2.12-2.1.1.tgz -C /usr/local
$ cd /usr/local/kafka_2.12-2.1.1/
##zk配置
$ vi config/zookeeper.properties
dataDir=/opt/kafka/zkdata
dataLogDir=/opt/kafka/zklog
clientPort=2181
maxClientCnxns=100
tickTime=2000
initLimit=10
syncLimit=5
server.1=192.168.11.97:2888:3888
server.2=192.168.11.98:2888:3888
server.3=192.168.11.99:2888:3888

mkdir /opt/kafka/zkdata -p
mkdir /opt/kafka/zklog -p
echo "1" > /opt/kafka/zkdata/myid
echo "2" > /opt/kafka/zkdata/myid
echo "3" > /opt/kafka/zkdata/myid

./bin/zookeeper-server-start.sh  config/zookeeper.properties &
```

## kafka配置
```
$ cd /usr/local/kafka_2.12-2.1.1
$ vi config/server.properties 
broker.id=1 # 1 2 3
listeners=PLAINTEXT://ip地址:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
log.dirs=/opt/kafka/kfklog
zookeeper.connect=192.168.11.97:2181,192.168.11.98:2181,192.168.11.99:2181
zookeeper.connection.timeout.ms=6000

$ mkdir /opt/kafka/kfklog
$ bin/kafka-server-start.sh -daemon config/server.properties
```

## 参数详解
```
# ----------------------系统相关----------------------
# broker的全局唯一编号，不能重复，和zookeeper的myid是一个意思
broker.id=0

# broker监听IP和端口也可以是域名
listeners=PLAINTEXT://172.16.48.163:9092

# 用于接收请求的线程数量
num.network.threads=3

# 用于处理请求的线程数量，包括磁盘IO请求，这个数量和log.dirs配置的目录数量有关，这里的数量不能小于log.dirs的数量，
# 虽然log.dirs是配置日志存放路径，但是它可以配置多个目录后面用逗号分隔
num.io.threads=8

# 发送缓冲区大小，也就是说发送消息先发送到缓冲区，当缓冲区满了之后一起发送出去
socket.send.buffer.bytes=102400

# 接收缓冲区大小，同理接收到缓冲区，当到达这个数量时就同步到磁盘
socket.receive.buffer.bytes=102400

# 向kafka套接字请求最大字节数量，防止服务器OOM，也就是OutOfMemery，这个数量不要超过JAVA的堆栈大小，
socket.request.max.bytes=104857600

# 日志路径也就是分区日志存放的地方，你所建立的topic的分区就在这里面，但是它可以配置多个目录后面用逗号分隔
log.dirs=/tmp/kafka-logs

# 消息体（也就是往Kafka发送的单条消息）最大大小，单位是字节，必须小于socket.request.max.bytes值
message.max.bytes =5000000

# 自动平衡由于某个broker故障会导致Leader副本迁移到别的broker，当之前的broker恢复后也不会迁移回来，有时候我们需要
# 手动进行平衡避免同一个主题不同分区的Leader副本在同一台broker上，下面这个参数就是开启自动平衡功能
auto.leader.rebalance.enable=true

# 设置了上面的自动平衡，当故障转移后，隔300秒（默认）触发一个定时任务进行平衡操作，而只有代理的不均衡率为10%以上才会执行
leader.imbalance.check.interval.seconds=300

# 设置代理的不均衡率，默认是10%
leader.imbalance.per.broker.percentage=10

# ---------------分区相关-------------------------

# 默认分区数量，当建立Topic时不指定分区数量，默认就1
num.partitions=1

# 是否允许自动创建topic ，若是false，就需要通过命令创建topic
auto.create.topics.enable =true
 
# 一个topic ，默认分区的replication个数 ，不得大于集群中broker的个数
default.replication.factor =2

# ---------------日志相关-------------------------

# segment文件默认会被保留7天的时间，超时的话就会被清理，那么清理这件事情就需要有一些线程来做。
# 这里就是用来设置恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1

# 日志文件中每个segment的大小，默认为1G。topic的分区是以一堆segment文件存储的，这个控制每个segment的大小，当超过这个大小会建立一个新日志文件
# 这个参数会被topic创建时的指定参数覆盖，如果你创建Topic的时候指定了这个参数，那么你以你指定的为准。
log.segment.bytes=1073741824

# 数据存储的最大时间 超过这个时间 会根据log.cleanup.policy设置的策略处理数据，也就是消费端能够多久去消费数据
# log.retention.bytes和log.retention.minutes|hours任意一个达到要求，都会执行删除
# 如果你创建Topic的时候指定了这个参数，那么你以你指定的为准
log.retention.hours|minutes=168

# 这个参数会在日志segment没有达到log.segment.bytes设置的大小默认1G的时候，也会强制新建一个segment会被
# topic创建时的指定参数覆盖
log.roll.hours=168 

# 上面的参数设置了每一个segment文件的大小是1G，那么就需要有一个东西去定期检查segment文件有没有达到1G，多长时间去检查一次，
# 就需要设置一个周期性检查文件大小的时间（单位是毫秒）。
log.retention.check.interval.ms=300000

# 日志清理策略 选择有：delete和compact 主要针对过期数据的处理，或是日志文件达到限制的额度，
# 如果你创建Topic的时候指定了这个参数，那么你以你指定的为准
log.cleanup.policy = delete

# 是否启用日志清理功能，默认是启用的且清理策略为compact，也就是压缩。
log.cleaner.enable=false

# 日志清理时所使用的缓存空间大小
log.cleaner.dedupe.buffer.size=134217728

# log文件"sync"到磁盘之前累积的消息条数，因为磁盘IO操作是一个慢操作,但又是一个"数据可靠性"的必要手段
# 所以此参数的设置,需要在"数据可靠性"与"性能"之间做必要的权衡.
# 如果此值过大,将会导致每次"fsync"的时间较长(IO阻塞)
# 如果此值过小,将会导致"fsync"的次数较多,这也意味着整体的client请求有一定的延迟.
# 物理server故障,将会导致没有fsync的消息丢失.
log.flush.interval.messages=9223372036854775807

# 检查是否需要固化到硬盘的时间间隔
log.flush.scheduler.interval.ms =3000
 
# 仅仅通过interval来控制消息的磁盘写入时机,是不足的.
# 此参数用于控制"fsync"的时间间隔,如果消息量始终没有达到阀值,但是离上一次磁盘同步的时间间隔
# 达到阀值,也将触发.
log.flush.interval.ms = None

# --------------------------复制(Leader、replicas) 相关-------------------
# partition leader与replicas之间通讯时,socket的超时时间
controller.socket.timeout.ms =30000

# replicas响应partition leader的最长等待时间，若是超过这个时间，就将replicas列入ISR(in-sync replicas)，
# 并认为它是死的，不会再加入管理中
replica.lag.time.max.ms =10000

# follower与leader之间的socket超时时间
replica.socket.timeout.ms=300000

# leader复制时候的socket缓存大小
replica.socket.receive.buffer.bytes=65536

# replicas每次获取数据的最大大小
replica.fetch.max.bytes =1048576

# replicas同leader之间通信的最大等待时间，失败了会重试
replica.fetch.wait.max.ms =500

# fetch的最小数据尺寸,如果leader中尚未同步的数据不足此值,将会阻塞,直到满足条件
replica.fetch.min.bytes =1

# leader 进行复制的线程数，增大这个数值会增加follower的IO
num.replica.fetchers=1

# 最小副本数量
min.insync.replicas = 2
```

## 一些命令
```
bin/kafka-topics.sh --create --zookeeper 192.168.11.97:2181,192.168.11.98:2181,192.168.11.99:2181 --replication-factor 2 --partitions 1 --topic test
bin/kafka-console-producer.sh --broker-list 192.168.11.97:9092 --topic test
bin/kafka-console-consumer.sh --boostrap-server 192.168.11.97:9092 --topic test --from-beginning
bin/kafka-topics.sh --list --zookeeper 192.168.11.97:2181,192.168.11.98:2181,192.168.11.99:2181
bin/kafka-topics.sh --describe --zookeeper 192.168.11.99:2181 
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --topic maxwell --partitions 1 --broker-list 192.168.11.99:9092

```



