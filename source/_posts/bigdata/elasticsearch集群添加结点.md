---
title: 添加elasticsearch结点||关闭elasticsearch集群
date: 2018-12-30 00:00:00
tags: 
- elasticsearch
categories: 
- tool
---
>elasticsearch 集群使用
<!-- more -->
1. 配置linux的elasticsearch环境
2. copy集群中的一个elasticsearch软件，作为新节点的elasticsearch软件
3. 修改elasticsearch.yaml,修改集群的配置包含新添加的结点。如
```yaml
[els@localhost elasticsearch-6.5.4]$ sed -n '/^[^#]/p' config/elasticsearch.yml  
cluster.name: logdig
node.name: node4
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
network.host: 192.168.106.237
discovery.zen.ping.unicast.hosts: ["192.168.106.213", "192.168.106.178","192.168.106.219","192.168.106.237"]
discovery.zen.minimum_master_nodes: 3
```
4. 关闭集群
```
[els@localhost elasticsearch-6.5.4]$ curl -XPUT http://192.168.106.213:9200/_cluster/settings -d '{
	"persistent":{
		"cluster.routing.allocation.enable":"none"
	}
}'
```
> 关闭集群的所有结点
```
$ kill -15 elspid
```
5. 启动集群包括新添加的结点
```
[es@localhost elasticsearch-6.5.3]$ bin/elasticsearch -d
[els@localhost elasticsearch-6.5.4]$ curl -XPUT http://192.168.106.213:9200/_cluster/settings -d '{
	"persistent":{
		"cluster.routing.allocation.enable":"all"
	}
}'
```
