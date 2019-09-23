---
title: hadoop配置说明
tags:
  - hadoop
categories:
  - db
date: 2019-08-22 02:00:00

---


> hadoop配置说明
<!-- more -->

## ResourceManager相关配置参数

* yarn.resourcemanager.address

> 参数解释：ResourceManager 对客户端暴露的地址。客户端通过该地址向RM提交应用程序，杀死应用程序等。

> 默认值：${yarn.resourcemanager.hostname}:8032

* yarn.resourcemanager.scheduler.address

> 参数解释：ResourceManager 对ApplicationMaster暴露的访问地址。ApplicationMaster通过该地址向RM申请资源、释放资源等。

> 默认值：${yarn.resourcemanager.hostname}:8030

* yarn.resourcemanager.resource-tracker.address

> 参数解释：ResourceManager 对NodeManager暴露的地址.。NodeManager通过该地址向RM汇报心跳，领取任务等。

> 默认值：${yarn.resourcemanager.hostname}:8031

* yarn.resourcemanager.admin.address

> 参数解释：ResourceManager 对管理员暴露的访问地址。管理员通过该地址向RM发送管理命令等。

> 默认值：${yarn.resourcemanager.hostname}:8033

* yarn.resourcemanager.webapp.address

> 参数解释：ResourceManager对外web ui地址。用户可通过该地址在浏览器中查看集群各类信息。

> 默认值：${yarn.resourcemanager.hostname}:8088

* yarn.resourcemanager.scheduler.class

> 参数解释：启用的资源调度器主类。目前可用的有FIFO、Capacity Scheduler和Fair Scheduler。

> 默认值：org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler

*  yarn.resourcemanager.resource-tracker.client.thread-count

> 参数解释：处理来自NodeManager的RPC请求的Handler数目。

> 默认值：50

* yarn.resourcemanager.scheduler.client.thread-count

> 参数解释：处理来自ApplicationMaster的RPC请求的Handler数目。

> 默认值：50

* yarn.scheduler.minimum-allocation-mb/ yarn.scheduler.maximum-allocation-mb

> 参数解释：单个可申请的最小/最大内存资源量。比如设置为1024和3072，则运行MapRedce作业时，每个Task最少可申请1024MB内存，最多可申请3072MB内存。

> 默认值：1024/8192

* yarn.scheduler.minimum-allocation-vcores / yarn.scheduler.maximum-allocation-vcores

> 参数解释：单个可申请的最小/最大虚拟CPU个数。比如设置为1和4，则运行MapRedce作业时，每个Task最少可申请1个虚拟CPU，最多可申请4个虚拟CPU。什么是虚拟CPU，可阅读我的这篇文章：“YARN 资源调度器剖析”。

> 默认值：1/32

* yarn.resourcemanager.nodes.include-path /yarn.resourcemanager.nodes.exclude-path

> 参数解释：NodeManager黑白名单。如果发现若干个NodeManager存在问题，比如故障率很高，任务运行失败率高，则可以将之加入黑名单中。注意，这两个配置参数可以动态生效。（调用一个refresh命令即可）

> 默认值：“”

* yarn.resourcemanager.nodemanagers.heartbeat-interval-ms

> 参数解释：NodeManager心跳间隔

> 默认值：1000（毫秒）

## NodeManager相关配置参数

* yarn.nodemanager.resource.memory-mb

> 参数解释：NodeManager总的可用物理内存。注意，该参数是不可修改的，一旦设置，整个运行过程中不可动态修改。另外，该参数的默认值是8192MB，即使你的机器内存不够8192MB，YARN也会按照这些内存来使用（傻不傻？），因此，这个值通过一定要配置。不过，Apache已经正在尝试将该参数做成可动态修改的。

> 默认值：8192

*  yarn.nodemanager.vmem-pmem-ratio

> 参数解释：每使用1MB物理内存，最多可用的虚拟内存数。

> 默认值：2.1

* yarn.nodemanager.resource.cpu-vcores

> 参数解释：NodeManager总的可用虚拟CPU个数。

> 默认值：8

* yarn.nodemanager.local-dirs

> 参数解释：中间结果存放位置，类似于1.0中的mapred.local.dir。注意，这个参数通常会配置多个目录，已分摊磁盘IO负载。

> 默认值：${hadoop.tmp.dir}/nm-local-dir

*  yarn.nodemanager.log-dirs

> 参数解释：日志存放地址（可配置多个目录）。

> 默认值：${yarn.log.dir}/userlogs

*  yarn.nodemanager.log.retain-seconds

> 参数解释：NodeManager上日志最多存放时间（不启用日志聚集功能时有效）。

> 默认值：10800（3小时）

* yarn.nodemanager.aux-services

> 参数解释：NodeManager上运行的附属服务。需配置成mapreduce_shuffle，才可运行MapReduce程序

> 默认值：“”


## 端口总结

端口 | 参数
-|-
9000|fs.defaultFS，如：hdfs://172.25.40.171:9000
9001|dfs.namenode.rpc-address，DataNode会连接这个端口
50070|	dfs.namenode.http-address
50470|	dfs.namenode.https-address
50100|	dfs.namenode.backup.address
50105|	dfs.namenode.backup.http-address
50090|	dfs.namenode.secondary.http-address，如：172.25.39.166:50090
50091|	dfs.namenode.secondary.https-address，如：172.25.39.166:50091
50020|	dfs.datanode.ipc.address
50075|	dfs.datanode.http.address
50475|	dfs.datanode.https.address
50010|	dfs.datanode.address，DataNode的数据传输端口
8480|	dfs.journalnode.rpc-address
8481|	dfs.journalnode.https-address
8032|	yarn.resourcemanager.address
8088|	yarn.resourcemanager.webapp.address，YARN的http端口
8090|	yarn.resourcemanager.webapp.https.address
8030|	yarn.resourcemanager.scheduler.address
8031|	yarn.resourcemanager.resource-tracker.address
8033|	yarn.resourcemanager.admin.address
8042|	yarn.nodemanager.webapp.address
8040|	yarn.nodemanager.localizer.address
8188|	yarn.timeline-service.webapp.address
10020|	mapreduce.jobhistory.address
19888|	mapreduce.jobhistory.webapp.address
2888|	ZooKeeper，如果是Leader，用来监听Follower的连接
3888|	ZooKeeper，用于Leader选举
2181|	ZooKeeper，用来监听客户端的连接
60010|	hbase.master.info.port，HMaster的http端口
60000|	hbase.master.port，HMaster的RPC端口
60030|	hbase.regionserver.info.port，HRegionServer的http端口
60020|	hbase.regionserver.port，HRegionServer的RPC端口
8080|	hbase.rest.port，HBase REST server的端口
10000|	hive.server2.thrift.port
9083|	hive.metastore.uris

## Hadoop配置文件的层级关系
  在Hadoop源码Configuration类中，先后加载了core-default.xml 和 core-site.xml ;类HdfsConfiguration继承了Configuration类，因此，hdfs-site.xml中的配置如果在core-site.xml 中配置了，那么将覆盖掉这个配置。 同理，yarn-site.xml也是这样的。
## yarn-site.xml 重点配置讲解
### 与 memery 相关的配置
* yarn.nodemanager.resource.memery-mb : 重要
 
>指定NodeManager（NM）可用的内存大小，NM是yarn中的一个组件，是一个Jave服务。这内存值是指NM管理的，用于AppMaster和Map Task 和 Reduce Task 可申请的内存数目。这里主要考虑和Linux系统的协调，避免设置过大被Linux的（Out of Memary Killer）OOMK 杀掉。例如2G的内存，设置为1200（1.2G）是可行的。 
默认配置被设置为8G，Hadoop就任务NM上就有8G的内存，因此当运行内存过大时，就有可能被linux被OOMK终结。 

* yarn.scheduler.minimum-allocation-mb: 

> 单个任务(每个Container)可申请最少内存，默认1024MB , 就是单个任务要申请的资源的话，ResourceManager(RM)会至少给你分配的内存数，即使你申请了1MB，也会派给你1024MB，因此这个地方需要根据机器配置和作业需求提前配置。 

* yarn.scheduler.increment-allocation-mb 

> 当一个Container拿到的最小内存不足以运行时，他可以申请内存递增，这个配置是配的递增的幅度。 

* yarn.scheduler.maximum-allocation-mb 

> 每个Container最多拿到的内存的数量，这里设置的是一个上限，Container申请资源时可递增直到满足运行需求，但是如果到了上限，就不能递增了。 

> 例：若yarn.scheduler.minimum-allocation-mb是50，yarn.scheduler.increment-allocation-mb是40，当申请60mb的时候，实际申请是80（50 < 60 < 40 * 2 ），当申请30mb的时候，实际是50（30 < 50，就是小于min）。


### 与CPU相关的配置

与上节节相似，也可以配置CPU的相应信息： 

* yarn.scheduler.minimum-allocation-vcores 
* yarn.scheduler.increment-allocation-vcores 
* yarn.scheduler.maximum-allocation-vcores


## mapred-site.xml 重点配置讲解
### MR AppMaster 相关

* yarn.app.mapreduce.am.resource.mb 
* yarn.app.mapreduce.am.resource.cpu-vcores 

> 这里am指 Yarn中AppMaster，针对MapReduce计算框架就是MR AppMaster，通过配置这两个选项，可以设定MR AppMaster使用的内存和cpu数量。 默认情况下，yarn.app.mapreduce.am.resource.mb是2G，因此这里必须配置，需要少于yarn.scheduler.minimum-allocation-mb的配置，这样才能成功启动。

### map、reduce task 相关

* mapreduce.map.memory.mb、mapreduce.map.cpu.vcores、mapreduce.reduce.memory.mb、mapreduce.reduce.cpu.vcores 
> 配置每个map 和 reduce 最多可以申请的内存和cpu数量;如果是单机情况，这里的资源和MR AppMaster 时共存和竞争关系，且都应在yarn.scheduler.minimum-allocation-mb之下，MR AppMaster启动成功后，剩下的资源才用于map、reduce task。

##  JVM 启动时参数设置

* mapreduce.map.java.opts、mapreduce.reduce.java.opts 
> Java task启动时使用的参数;例如设置Java heap 的大小，会在启动map或者reduce进程时把这些参数附加在启动命令之后。 
这个地方需要仔细配置，配置不当会引起JVM 不能正常GC。 


例如：

- mapreduce.map.memory.mb 设置为了500MB
- mapreduce.map.java.opts 设置为 -Xmx600m
- 假设CG为JVM中内存占用达到90%时触发，也就是540MB时会触发
这时，由于mapreduce.map.memory.mb的限制，map 进程会被提前Kill掉，因此GC永远得不到触发。

## core-site.xml

* fs.defaultFS（老版本是fs.default.name） 

> 默认使用的文件系统类型（hdfs://host1:8020/、viewfs://nsX）

* fs.trash.interval 

> 垃圾箱文件保留多久（单位：分钟）

* io.compression.codecs 

> Java codec

* hadoop.security.authentication 

> Hadoop使用的认证方法（simple或kerberos）

## hdfs-site.xml

• dfs.namenode.name.dir 
– namenode存放fsimage的目录

• dfs.datanode.data.dir 
– datanode存放数据块文件的目录

• dfs.namenode.checkpoint.dir 
– Secondarynamenode启动时使用，放置sn做合并的fsimage及 editlog

• dfs.replication 
– 数据副本数

• dfs.blocksize 
– 文件Block大小

• dfs.datanode.handler.count 
– Datanode IPC 请求处理线程数

##  yarn-site.xml其他配置

* yarn.resourcemanager.hostname、yarn.resourcemanager.address、yarn.resourcemanager.admin.address 
、yarn.resourcemanager.scheduler.address、 yarn.resourcemanager.resource-tracker.address、 
yarn.resourcemanager.webapp.address等项目的基本，依据实际rm 的hostname配置

* yarn.scheduler.minimum-allocation-mb 

> Yarn分配内存的最小单位

* yarn.scheduler.increment -allocation-mb 

> 内存分配递增最小单位

* yarn.scheduler.maximum-allocation-mb 

> 每个container最多申请的内存上限

* yarn.scheduler.minimum-allocation-vcores、 
yarn.scheduler.increment -allocation-vcores、 
yarn.scheduler.maximum-allocation-vcores 

> 跟以上内存相关的意义相同

* yarn.resourcemanager.am.max-retries 

> AM失败重试次数

* yarn.am.liveness-monitor.expiry-interval-ms 

> AM多久不上报心跳被视为过期

* yarn.resourcemanager.nm.liveness-monitor.interval-ms 

> 多久检查一次nm是否还存活

* yarn.application.classpath 

> Yarn application使用的classpath

* yarn.resourcemanager.scheduler.class 

> 调度器定义，如 org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler

* yarn.scheduler.fair.user -as-default -queue 

> 对于fairscheduler使用，是否使用用户名作为任务进入的queue

* yarn.scheduler.fair.preemption 

> Fairescheduler是否支持queue之间任务抢占

* yarn.resourcemanager.max-completed-applications 

> RM中保留的最大的已完成的任务信息数量

* yarn.nodemanager.resource.memory-mb 

> Nm的可用内存

* yarn.nodemanager.local-dirs 

> Nm本地文件目录（主要是一些cache)

* yarn.nodemanager.log-dirs 

> Nm log目录

* yarn.nodemanager.remote-app-log-dir、yarn.logaggregation.retain-seconds、yarn.log-aggregationenable、yarn.nodemanager.logaggregation.compression-type 

> 启动log aggregation的时候使用参数

* yarn.app.mapreduce.am.staging-dir 

> Staging目录，hdfs

## mapred-site.xml 其他配置讲解

* mapreduce.map.speculative、mapreduce.reduce.speculative 

> 是否开启预测执行

* mapreduce.job.reduce.slowstart.completedmaps 

> Map完成多少之后开始启动reduce

* mapreduce.jobhistory.address、mapreduce.jobhistory.webapp.address 

> Jobhistory地址

* mapreduce.framework.name 

> local, classic or yarn

* yarn.app.mapreduce.am.staging -dir、yarn.app.mapreduce.am.resource.mb 
、yarn.app.mapreduce.am.resource.cpu-vcores、 yarn.app.mapreduce.am.command-opts

> Am相关配置 

* mapreduce.map.java.opts、mapreduce.reduce.java.opts 

> Java task启动时使用的参数

* mapreduce.map.memory.mb、mapreduce.map.cpu.vcores、mapreduce.reduce.memory.mb、mapreduce.reduce.cpu.vcores 

> Map/reduce任务资源设置

* mapreduce.application.classpath 

> Task使用的classpath

## 问答

* yarn.scheduler.minimum-allocation-mb，为app分配内存时最小的配额。 
* yarn.scheduler.increment-allocation-mb，每次递加申请的内存资源数，比如，若yarn.scheduler.minimum-allocation-mb是50，yarn.scheduler.increment-allocation-mb是40，当申请60mb的时候，实际申请是80（50 < 60 < 40 * 2 ），当申请30mb的时候，实际是50（30 < 50，就是小于min）。 
* yarn.scheduler.maximum-allocation-mb，单个container可分配的内存总量上限。

* yarn.app.mapreduce.am.resource.mb，执行appManager时需要分配的内存。 
* mapreduce.map.memory.mb，map任务执行时要申请的内存。 
* mapreduce.reduce.memory.mb，reduce任务执行时要申请的内存。 
* mapreduce.task.io.sort.mb，任务在做spill时，内存的缓存量，之所以提出来，因为当我们将mapreduce.map.memory.mb和apreduce.reduce.memory.mb减小时，需要将这个值也减小，否则会出现task任务资源不够跑不成功的问题
