---
title: Maxwell配置详解
tags:
  - maxwell
categories:
  - db
date: 2019-05-29 03:00:00

---


> Maxwell配置详解
<!-- more -->


> mysql options
* host
指定从哪个地址的mysql获取binlog
* replication_host
如果指定了 replication_host，那么它是真正的binlog来源的mysql server地址，而那么上面的host用于存放maxwell表结构和binlog位置的地址。
将两者分开，可以避免 replication_user 往生产库里写数据。
* schema_host
从哪个host获取表结构。binlog里面没有字段信息，所以maxwell需要从数据库查出schema，存起来。
schema_host一般用不到，但在binlog-proxy场景下就很实用。比如要将已经离线的binlog通过maxwell生成json流，于是自建一个mysql server里面没有结构，只用于发送binlog，此时表机构就可以制动从 schema_host 获取。

* gtid_mode
如果 mysql server 启用了GTID，maxwell也可以基于gtid取event。如果mysql server发生failover，maxwell不需要手动指定newfile:postion

正常情况下，replication_host 和 schema_host都不需要指定，只有一个 --host。

* schema_database
使用这个db来存放 maxwell 需要的表，比如要复制的databases, tables, columns, postions, heartbeats.




> filtering
* include_dbs
只发送binlog里面这些databases的变更，以,号分隔，中间不要包含空格。
也支持java风格的正则，如 include_tables=db1,/db\\d+/，表示 db1, db2, db3…这样的。（下面的filter都支持这种regex）
提示：这里的dbs指定的是真实db。比如binlog里面可能 use db1 但 update db2.ttt，那么maxwell生成的json database 内容是db2。
* exclude_dbs
排除指定的这些 databbases
* include_tables
只发送这些表的数据变更。不只需要指定 database.
* exclude_tables
排除指定的这些表
* exclude_columns
不输出这些字段。如果字段名在row中不存在，则忽略这个filter。
* include_column_values
1.12.0新引入的过滤项。只输出满足 column=values 的行，比如 include_column_values=bar=x,foo=y，如果有bar字段，那么只输出值为x的行，如果有foo字段，那么只输出值为y的行。
如果没有对应字段，如只有bar=x没有foo字段，那么也成立。（即不是 或，也不是 与）

* blacklist_dbs
一般不用。blacklist_dbs字面上难以与exclude_dbs 分开，官网的说明也是模棱两可。
从代码里面看出的意思是，屏蔽指定的这些dbs,tables的结构变更，与行变更过滤，没有关系。它应对的场景是，某个表上频繁的有ddl，比如truncate。










> formatting
* output_ddl
是否在输出的json流中，包含ddl语句。默认 false
* output_binlog_position
是否在输出的json流中，包含binlog filename:postion。默认 false
* output_commit_info
是否在输出的json流里面，包含 commit 和 xid 信息。默认 true
比如一个事物里，包含多个表的变更，或一个表上多条数据的变更，那么他们都具有相同的 xid，最后一个row event输出 commit:true 字段。这有利于消费者实现 事务回放，而不仅仅是行级别的回放。
* output_thread_id
同样，binlog里面也包含了 thread_id ，可以包含在输出中。默认 false
消费者可以用它来实现更粗粒度的事务回放。还有一个场景是用户审计，用户每次登陆之后将登陆ip、登陆时间、用户名、thread_id记录到一个表中，可轻松根据thread_id关联到binlog里面这条记录是哪个用户修改的。
* monitoring
如果是长时间运行的maxwell，添加monitor配置，maxwell提供了http api返回监控数据。

> 其它
* init_position
手动指定maxwell要从哪个binlog，哪个位置开始。指定的格式FILE:POSITION:HEARTBEAT。只支持在启动maxwell的命令指定，比如 --init_postion=mysql-bin.0000456:4:0。
maxwell 默认从连接上mysql server的当前位置开始解析，如果指定 init_postion，要确保文件确实存在，如果binlog已经被purge掉了，可能需要想其它办法。见 Binlog可视化搜索：实现类似阿里RDS数据追踪功能