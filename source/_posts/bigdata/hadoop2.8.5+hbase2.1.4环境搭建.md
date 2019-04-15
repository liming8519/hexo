---
title: hadoop2.8.5+hbase2.1.4+spark2.3.3环境搭建
tags:
  - hadoop
categories:
  - db
date: 2019-04-04 02:00:00

---


> hadoop2.8.5+hbase2.1.4环境搭建
<!-- more -->

## centos7.3 配置
* 119设置
```
[root@localhost ~]# hostnamectl set-hostname hadoopa
[root@hadoopa ~]# vi /etc/hosts
192.168.11.119 hadoopa
192.168.11.118 hadoopb
192.168.11.117 hadoopc
```
* 118设置
```
[root@localhost ~]# hostnamectl set-hostname hadoopb
[root@hadoopa ~]# vi /etc/hosts
192.168.11.119 hadoopa
192.168.11.118 hadoopb
192.168.11.117 hadoopc
```
* 117设置
```
[root@localhost ~]# hostnamectl set-hostname hadoopc
[root@hadoopa ~]# vi /etc/hosts
192.168.11.119 hadoopa
192.168.11.118 hadoopb
192.168.11.117 hadoopc
```
## java安装
* 分别把117jdk拷贝到三台机器的opt目录解压即可
```
#以下在117上执行
[root@hadoopc opt]# scp jdk-8u201-linux-x64.tar.gz 192.168.11.118:/opt
#以下在117 118 119上分别执行
[root@hadoopc opt]# tar -xzvf jdk-8u201-linux-x64.tar.gz
[root@hadoopc opt]# mv jdk1.8.0_201/ jdk1.8
```
## ssh免密码配置
* 117上执行
```
[root@hadoopc opt]# cd ~/.ssh/
[root@hadoopc .ssh]# ssh-keygen -t rsa #生成密钥和pub公钥
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
36:20:71:94:09:e4:1e:94:19:61:01:8c:e6:ea:e3:c4 root@hadoopc
The key's randomart image is:
+--[ RSA 2048]----+
| o.oXOoo         |
|...+ooo          |
|o   + .          |
| . . o .         |
|.   .   S        |
|o      . .       |
|.E               |
|.o               |
|...              |
+-----------------+
[root@hadoopc .ssh]# ls
id_rsa  id_rsa.pub  known_hosts
[root@hadoopc .ssh]# cat id_rsa.pub >> authorized_keys #此时会生成authorized_keys文件，文件中存放的是你想免密码登录的pub公钥，当前命令说明当执行ssh localhost免密码，118  119免密码登录同理把在118 和 119 下生成的公钥拷贝到当前authorized_keys文件中。
```
* 118上执行
```
[root@hadoopc opt]# cd ~/.ssh/
[root@hadoopc .ssh]# ssh-keygen -t rsa
[root@hadoopc .ssh]# cat id_rsa.pub >> authorized_keys
```
* 119上执行
```
[root@hadoopc opt]# cd ~/.ssh/
[root@hadoopc .ssh]# ssh-keygen -t rsa
[root@hadoopc .ssh]# cat id_rsa.pub >> authorized_keys
```
* 分别把118和119上的pub公钥拷贝到117的authorized_keys文件中，要用scp不能直接复制
* 分别把117的authorized_keys文件拷贝到118和119中替换他们的authorized_keys文件
* ssh 登录验证一下，看是否还需要密码

## 安装hadoop

### 以下配置仅在117上执行

#### hdoop配置
```
#仅在117上执行
[root@hadoopa ~]# cd /opt
[root@hadoopc opt]# ls
hadoop-2.8.5.tar.gz  jdk1.8
[root@hadoopc opt]# mkdir hadoop285
[root@hadoopc opt]# tar -zxvf /opt/hadoop-2.8.5.tar.gz -C /opt/hadoop285
[root@hadoopc opt]# cd /opt/hadoop285/hadoop-2.8.5/etc/hadoop
[root@hadoopc hadoop]# vi hadoop-env.sh 
#export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/opt/jdk1.8
#export HADOOP_HOME=/opt/hadoop285/hadoop-2.8.5
#export PATH=$HADOOP_HOME/bin:$JAVA_HOME/bin:$PATH
[root@hadoopc hadoop]# vi core-site.xml 
<configuration>
  <property> 
    <name>hadoop.tmp.dir</name> 
    <value>/opt/hadoop285/tmp</value> 
    <description>A base for other temporary directories.</description> 
  </property> 
  <property> 
    <name>fs.default.name</name> 
    <value>hdfs://192.168.11.119:9000</value> 
  </property> 
　<property> 
　    <name>io.file.buffer.size</name> 
　    <value>4096</value> 
　 </property> 
</configuration>
[root@hadoopc hadoop]# vi hdfs-site.xml 
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value> 
  </property>
  <property>
    <name>dfs.name.dir</name> 
    <value>/opt/hadoop285/hdfs/name</value> 
  </property>
  <property>
    <name>dfs.data.dir</name> 
    <value>/opt/hadoop285/hdfs/data</value> 
  </property>
</configuration>
[root@hadoopc hadoop]# cp mapred-site.xml.template mapred-site.xml
[root@hadoopc hadoop]# vi mapred-site.xml
<configuration>
  <!--指定maoreduce运行框架--> 
  <property> 
    <name>mapreduce.framework.name</name> 
    <value>yarn</value>
  </property> 
  <!--历史服务的通信地址--> 
  <property> 
    <name>mapreduce.jobhistory.address</name> 
    <value>192.168.11.119:10020</value> 
  </property> 
  <!--历史服务的web ui地址--> 
  <property> 
    <name>mapreduce.jobhistory.webapp.address</name> 
    <value>192.168.11.119:19888</value> 
  </property>
</configuration>
[root@hadoopc hadoop]# vi yarn-site.xml 
<configuration>
<!-- Site specific YARN configuration properties -->
<!--指定resourcemanager所启动的服务器主机名--> 
  <property> 
    <name>yarn.resourcemanager.hostname</name> 
    <value>192.168.11.119</value> 
  </property> 
<!--指定mapreduce的shuffle--> 
  <property> 
    <name>yarn.nodemanager.aux-services</name> 
    <value>mapreduce_shuffle</value> 
  </property> 
<!--指定resourcemanager的内部通讯地址--> 
  <property> 
    <name>yarn.resourcemanager.address</name> 
    <value>192.168.11.119:8032</value> 
  </property> 
<!--指定scheduler的内部通讯地址--> 
  <property> 
    <name>yarn.resourcemanager.scheduler.address</name> 
    <value>192.168.11.119:8030</value> 
  </property> 
<!--指定resource-tracker的内部通讯地址--> 
  <property> 
    <name>yarn.resourcemanager.resource-tracker.address</name> 
    <value>192.168.11.119:8031</value> 
  </property> 
<!--指定resourcemanager.admin的内部通讯地址--> 
  <property> 
    <name>yarn.resourcemanager.admin.address</name> 
    <value>192.168.11.119:8033</value> 
  </property> 
<!--指定resourcemanager.webapp的ui监控地址--> 
  <property> 
    <name>yarn.resourcemanager.webapp.address</name> 
    <value>192.168.11.119:8088</value> 
  </property>
</configuration>
[root@hadoopc hadoop]# vi slaves 
192.168.11.117
192.168.11.118
[root@hadoopc hadoop]# vi yarn-env.sh
# some Java parameters
# export JAVA_HOME=/home/y/libexec/jdk1.6.0/
export JAVA_HOME=/opt/jdk1.8
[root@hadoopc hadoop]# vi mapred-env.sh
# export JAVA_HOME=/home/y/libexec/jdk1.6.0/
export JAVA_HOME=/opt/jdk1.8

```
#### 存放信息相关文件创建

```
[root@hadoopc hadoop285]# mkdir /opt/hadoop285/tmp
[root@hadoopc hadoop285]# mkdir /opt/hadoop285/hdfs
[root@hadoopc hadoop285]# mkdir /opt/hadoop285/hdfs/name
[root@hadoopc hadoop285]# mkdir /opt/hadoop285/hdfs/data
```
#### 把配置好的hadoop拷贝到118和119
```
[root@hadoopc hadoop]# cd /opt
[root@hadoopc opt]# scp -r hadoop285/ 192.168.11.118:`pwd`
[root@hadoopc opt]# scp -r hadoop285/ 192.168.11.119:`pwd`
```
## 启动hdoop
### 只是一个记录

```
#只是一个记录
export JAVA_HOME=/opt/jdk1.8
export HADOOP_HOME=/opt/hadoop285/hadoop-2.8.5
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin:$PATH
　　
```
### 启动

```
[root@hadoopa fk]# hdfs namenode -format
[root@hadoopa fk]# start-dfs.sh
[root@hadoopa fk]# start-yarn.sh

```
### 打开http://192.168.11.119:50070/验证


## zookeeper安装版本是3.4.14
### zookeeper仅安装到117上，非集群安装
#### 配置
```
[root@hadoopc opt]# pwd
/opt
[root@hadoopc opt]# mkdir zookeeper3414
[root@hadoopc opt]# tar -xzvf zookeeper-3.4.14.tar.gz -C /opt/zookeeper3414/
[root@hadoopc opt]# cd /opt/zookeeper3414/zookeeper-3.4.14/conf
[root@hadoopc conf]# cp zoo_sample.cfg zoo.cfg
[root@hadoopc conf]# vi zoo.cfg 
dataDir=/opt/zookeeper3414/data
dataLogDir=/opt/zookeeper3414/logs
[root@hadoopc conf]# mkdir /opt/zookeeper3414/data
[root@hadoopc conf]# mkdir /opt/zookeeper3414/logs
```
#### 标记
```
export PATH=/opt/zookeeper3414/zookeeper-3.4.14/bin:$PATH
```


#### 启动zookeeper
```
[root@hadoopc conf]# zkServer.sh start
[root@hadoopc conf]# zkServer.sh stop
[root@hadoopc conf]# zkServer.sh restart
[root@hadoopc conf]# zkServer.sh status
#QuorumPeerMain是zookeeper的进程名字
```


## HBASE安装

### 以下仅在117上执行
```
[root@hadoopc opt]# mkdir hbase214
[root@hadoopc opt]# tar -zxvf hbase-2.1.4-bin.tar.gz -C /opt/hbase214
[root@hadoopc opt]# cd /opt/hbase214/hbase-2.1.4/conf
[root@hadoopc conf]# vi hbase-env.sh
export HBASE_PID_DIR=/opt/hbase214/pids
export HBASE_MANAGES_ZK=false
export JAVA_HOME=/opt/jdk1.8
[root@hadoopc conf]# vi hbase-site.xml
<configuration>
  <!-- 指定hbase master机器 -->
  <property>
    <name>hbase.master</name>
    <value>hdfs://192.168.11.119:9000</value>
  </property>
  <!-- hbase分布式集群为true -->
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <!-- 指定ZK集群地址,多个用逗号分开-->
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>192.168.11.117</value>
  </property>
  <!-- 指定独立Zookeeper安装路径 -->
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/zookeeper3414/zookeeper-3.4.14</value>
  </property>
  <!-- 指定ZooKeeper集群端口 -->
  <property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <!--此处避免一个启动时候得错误-->
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
　<property>
　    <name>hbase.rootdir</name>
　    <value>hdfs://192.168.11.119:9000/hbase</value>
　  </property>
</configuration>
[root@hadoopc conf]# vi regionservers 
192.168.11.117
192.168.11.118
192.168.11.119
[root@hadoopc conf]# cd /opt/hbase214/hbase-2.1.4/lib/client-facing-thirdparty
[root@hadoopc client-facing-thirdparty]# cp htrace-core4-4.2.0-incubating.jar ../
```
### 标记

```
export PATH=/opt/hbase214/hbase-2.1.4/bin:$PATH
```
### 配置拷贝到118和119

```
[root@hadoopc opt]# cd /opt　　
[root@hadoopc opt]# scp -r hbase214/ 192.168.11.118:`pwd`
[root@hadoopc opt]# scp -r hbase214/ 192.168.11.119:`pwd`

```

### 启动hbase

```
[root@hadoopa conf]# start-hbase.sh
```
### 关于报错解决
* 报错信息是：Caused by: java.lang.ClassNotFoundException: org.apache.htrace.SamplerBuilder
* 进过百度说缺少htrace-core-3.1.0-incubating.jar包但hbase2.1.4的lib\client-facing-thirdparty目录下没有这个文件，解决方法是下载hbase2.1.3在目录lib\client-facing-thirdparty中找到文件htrace-core-3.1.0-incubating.jar拷贝到hbase2.1.4的lib目录中。

## spark安装（2.3.3）

### 以下仅在117上执行
```
[root@hadoopc ~]# cd /home
[root@hadoopc opt]# mkdir spark233
[root@hadoopc opt]# tar -xzvf /opt/spark-2.3.3-bin-hadoop2.7.tgz -C /home/spark233/
[root@hadoopc home]# cd spark233/spark-2.3.3-bin-hadoop2.7/
[root@hadoopc spark-2.3.3-bin-hadoop2.7]# cd conf
[root@hadoopc conf]# cp spark-env.sh.template spark-env.sh
[root@hadoopc conf]# vi spark-env.sh 
export JAVA_HOME=/opt/jdk1.8
export HADOOP_CONF_DIR=/opt/hadoop285/hadoop-2.8.5/etc/hadoop/
export YARN_CONF_DIR=/opt/hadoop285/hadoop-2.8.5/etc/hadoop/
export SPARK_MASTER_PORT=7077
export SPARK_HOME=/home/spark233/spark-2.3.3-bin-hadoop2.7
[root@hadoopc conf]# cp slaves.template slaves
[root@hadoopc conf]# vi slaves
#localhost
#192.168.11.117
#不需要指定MASTER在哪里启动哪里就是Master
192.168.11.118
192.168.11.119
[root@hadoopc conf]# cd ../../..
[root@hadoopc home]# scp -r spark233/ 192.168.11.118:`pwd`
[root@hadoopc home]# scp -r spark233/ 192.168.11.119:`pwd`
```

### 标记

```
export SPARK_HOME=/home/spark233/spark-2.3.3-bin-hadoop2.7
export PATH=$PATH:$SPARK_HOME/sbin:$SPARK_HOME/bin
```

### 启动
```
[root@hadoopc sbin]# ./start-master.sh 
[root@hadoopc sbin]# ./start-all.sh 
```
