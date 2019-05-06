title: elasticsearch 集群安装
- elasticsearch 
categories: 
- tool 
date: 2019-05-05 02:00:00

> elasticsearch 集群安装
<--! more -->

## 环境设置
> 与ELK文档一致
```
$ groupadd elk
$ useradd -g elk -m elk
$ passwd elk
$ vi /etc/security/limits.conf
 * soft nofile 65536
 * hard nofile 131072
 * soft nproc 2048
 * hard nproc 4096
 * soft memlock unlimited 
 * hard memlock unlimited 
$ vi /etc/security/limits.d/20-nproc.conf
* soft nproc 4096
$ vi /etc/sysctl.conf
vm.max_map_count=655360
$ sysctl -p
##java jdk
$ #以下是elk用户下
$ tar -xzvf jdk-8u192-linux-x64.tar.gz 
$ mv jdk1.8.0_192/ jdk
$ vi ~/.bash_profile
export JAVA_HOME=/home/elk/soft/jdk
PATH=$PATH:$HOME/.local/bin:$HOME/bin:/home/elk/soft/jdk/bin
$ source .bash_profile
```

## elasticsearch集群配置安装
```
$ tar -xzvf elasticsearch-6.6.2.tar.gz
$ mkdir /home/elk/soft/elasticsearch-6.6.2/data
##$ mkdir /home/elk/soft/elasticsearch-6.6.2/logs
$ cd elasticsearch-6.6.2/config/
$ vi elasticsearch.yml 

#集群的名称
cluster.name: ythtest
#节点名称,其余两个节点分别为node15 和node16
node.name: node14
#指定该节点是否有资格被选举成为master节点，默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master
node.master: true
#允许该节点存储数据(默认开启)
node.data: true
#索引数据的存储路径
path.data: /home/elk/soft/elasticsearch-6.6.2/data
#日志文件的存储路径
path.logs: /home/elk/soft/elasticsearch-6.6.2/logs
#设置为true来锁住内存。因为内存交换到磁盘对服务器性能来说是致命的，当jvm开始swapping时es的效率会降低，所以要保证它不swap
bootstrap.memory_lock: true
#绑定的ip地址
network.host: 0.0.0.0
#设置对外服务的http端口，默认为9200
http.port: 9200
# 设置节点间交互的tcp端口,默认是9300 
transport.tcp.port: 9300
#Elasticsearch将绑定到可用的环回地址，并将扫描端口9300到9305以尝试连接到运行在同一台服务器上的其他节点。
#这提供了自动集群体验，而无需进行任何配置。数组设置或逗号分隔的设置。每个值的形式应该是host:port或host
#（如果没有设置，port默认设置会transport.profiles.default.port 回落到transport.tcp.port）。
#请注意，IPv6主机必须放在括号内。默认为127.0.0.1, [::1]
discovery.zen.ping.unicast.hosts: ["192.168.11.14:9300", "192.168.11.15:9300", "192.168.11.16:9300"]
#如果没有这种设置,遭受网络故障的集群就有可能将集群分成两个独立的集群 - 分裂的大脑 - 这将导致数据丢失
discovery.zen.minimum_master_nodes: 3
#一个集群中的N个节点启动后,才允许进行数据恢复处理，默认是1
gateway.recover_after_nodes: 2

##启动elasticsearch
$ /home/elk/soft/elasticsearch-6.6.2/bin/elasticsearch -d
```
## 集群配置
```
PUT /_cluster/settings
{
    "persistent" : {
        "indices.store.throttle.max_bytes_per_sec" : "100mb"#普通磁盘建议64mb
        "indices.store.throttle.type" : "none" #不限制磁盘流量
        index.merge.scheduler.max_thread_count: 1
        index.refresh_interval: 30s
        index.number_of_replicas: 0
    }
}

PUT /_all/_settings 
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m" 
  }
}

PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "none"
    }
}

PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
}
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true" 
    }
}
```

## 配置总结

* cluster.name:
集群名字
* node.name:
结点名称
* path.data: /path/to/data1,/path/to/data2 
数据目录；多个数据路径只是帮助如果你有许多索引／分片在单个节点上
* path.logs: /path/to/logs
Path to log files

* path.plugins: /path/to/plugins
Path to where plugins are installed
* discovery.zen.minimum_master_nodes: 2
这个配置就是告诉 Elasticsearch 当没有足够 master 候选节点的时候，就不要进行 master 节点选举，等 master 候选节点足够了才进行选举。
此设置应该始终被配置为 master 候选节点的法定个数（大多数个）。法定个数就是 ( master 候选节点个数 / 2) + 1 。
```
PUT /_cluster/settings
{
    "persistent" : {
        "discovery.zen.minimum_master_nodes" : 2
    }
}
```
* gateway.recover_after_nodes: 8
这意味着至少要有 8 个节点，该集群才可用。
* gateway.expected_nodes: 10
* gateway.recover_after_time: 5m
结合上一条配置，说明：等待集群至少存在 8 个节点;等待 5 分钟，或者10 个节点上线后，才进行数据恢复，这取决于哪个条件先达到。

* discovery.zen.ping.unicast.hosts: ["host1", "host2:port"]
使用单播，你可以为 Elasticsearch 提供一些它应该去尝试连接的节点列表。 当一个节点联系到单播列表中的成员时，它就会得到整个集群所有节点的状态，然后它会联系 master 节点，并加入集群。这意味着你的单播列表不需要包含你的集群中的所有节点， 它只是需要足够的节点，当一个新节点联系上其中一个并且说上话就可以了。如果你使用 master 候选节点作为单播列表，你只要列出三个就可以了

* export ES_HEAP_SIZE=10g
* ./bin/elasticsearch -Xmx10g -Xms10g
设置内存为机器内存的一般，但不能超过32G，超过32G的话jvm会使用64位的指针，这样会浪费大量的内存。
* sudo swapoff -a
暂时禁用swap
* vm.swappiness = 1 
swappiness 设置为 1 比设置为 0 要好，因为在一些内核版本 swappiness 设置为 0 会触发系统 OOM-killer（注：Linux 内核的 Out of Memory（OOM）killer 机制）。
* bootstrap.mlockall: true
如果上面的方法都不合适，你需要打开配置文件中的 mlockall 开关。 它的作用就是允许 JVM 锁住内存，禁止操作系统交换出去。在你的 elasticsearch.yml 文件中设置。
* sysctl -w vm.max_map_count=262144
在 /etc/sysctl.conf 通过修改 vm.max_map_count 永久设置它
* PUT /my_index/_settings
{
    "index.search.slowlog.threshold.query.warn" : "10s", 
    "index.search.slowlog.threshold.fetch.debug": "500ms", 
    "index.indexing.slowlog.threshold.index.info": "5s" 
}
查询慢于 10 秒输出一个 WARN 日志。
获取慢于 500 毫秒输出一个 DEBUG 日志。
索引慢于 5 秒输出一个 INFO 日志。

* PUT /_cluster/settings
{
    "persistent" : {
        "indices.store.throttle.max_bytes_per_sec" : "100mb"#机械磁盘可以设置60mb
    }
}
"indices.store.throttle.type" : "none" ：关闭限流

* index.merge.scheduler.max_thread_count: 1
机械磁盘在并发 I/O 支持方面比较差，所以我们需要降低每个索引并发访问磁盘的线程数。这个设置允许 max_thread_count + 2 个线程同时进行磁盘操作，也就是设置为 1 允许三个线程。
* 如果你的搜索结果不需要近实时的准确度，考虑把每个索引的 index.refresh_interval 改到 30s
* 如果你在做大批量导入，考虑通过设置 index.number_of_replicas: 0关闭副本。
* PUT /_all/_settings 
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m" 
  }
}
通过使用 _all 索引名，我们可以为集群里面的所有的索引使用这个参数
默认时间被修改成了 5 分钟
默认情况，集群会等待一分钟来查看节点是否会重新加入，如果这个节点在此期间重新加入，重新加入的节点会保持其现有的分片数据，不会触发新的分片分配。
* 滚动重启
1.可能的话，停止索引新的数据。虽然不是每次都能真的做到，但是这一步可以帮助提高恢复速度。
2.禁止分片分配。这一步阻止 Elasticsearch 再平衡缺失的分片，直到你告诉它可以进行了。如果你知道维护窗口会很短，这个主意棒极了。你可以像下面这样禁止分配：
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "none"
    }
}
3.关闭单个节点。
4.执行维护/升级。
5.重启节点，然后确认它加入到集群了。
6.用如下命令重启分片分配：
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
}
分片再平衡会花一些时间。一直等到集群变成 绿色 状态后再继续。
7.重复第 2 到 6 步操作剩余节点。
8.到这步你可以安全的恢复索引了（如果你之前停止了的话），不过等待集群完全均衡后再恢复索引，也会有助于提高处理速度。
