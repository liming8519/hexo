---
title: Mysql主从搭建
tags:
  - mysql
categories:
  - db
date: 2019-01-11 00:00:00
---
> mysql复制搭建过程
<!-- more -->
## 关键配置参数
```
---master----
server-id=1
log-bin=master-bin
---slaver----
server-id=2
log-bin=slavera-bin
log-slave-updates
```
## 备份master
```
innobackupex --user=root --password=root --slaver-info
--database=dataprocessno1_bigdata /root/bak
```
## 打包备份
```
tar -zcvf bak.tar.gz bak
```
## 备份拷贝到目标机器
```
scp bak.tar.gz 192.168.106.174:/root/bak
```
## Slaver还原
```
tar -xzvf bak.tar.gz
cd bak
mv * ../
innobackupex --user=root --apply-log /root/bak/2018-01-10_16-09-46
innobackupex --user=root --copy-back /root/bak/2018-01-10_16-09-46
chown mysql:mysql -R /usr/local/mysql/data
```
## Master创建复制帐号
```
mysql> grant replication slave on *.* to repl@'192.168.106.%' identified by
'repl';
```
## 启动slaver并指向master
```
[root@oracle 2018-01-10_16-09-46]# cat xtrabackup_binlog_info
master-bin.000001 154
mysql>change master to
master_host='192.168.106.234',master_user='repl',master_password='repl',master_log_file='master-bin.000001',master_log_pos=154;
mysql>start slave;
mysql>show slave status\G
Slave_IO_State: Waiting for master to send event
Master_Host: 192.168.106.234
Master_User: repl
Master_Port: 3306
Connect_Retry: 60
Master_Log_File: master-bin.000001
Read_Master_Log_Pos: 593
Relay_Log_File: oracle-relay-bin.000002
Relay_Log_Pos: 760
Relay_Master_Log_File: master-bin.000001
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Replicate_Do_DB:
Replicate_Ignore_DB:
Replicate_Do_Table:
Replicate_Ignore_Table:
Replicate_Wild_Do_Table:
Replicate_Wild_Ignore_Table:
Last_Errno: 0
Last_Error:
Skip_Counter: 0
Exec_Master_Log_Pos: 593
Relay_Log_Space: 968
Until_Condition: None
Until_Log_File:
Until_Log_Pos: 0
Master_SSL_Allowed: No
Master_SSL_CA_File:
Master_SSL_CA_Path:
Master_SSL_Cert:
Master_SSL_Cipher:
Master_SSL_Key:
Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
Last_IO_Errno: 0
Last_IO_Error:
Last_SQL_Errno: 0
Last_SQL_Error:
Replicate_Ignore_Server_Ids:
Master_Server_Id: 1
Master_UUID: a34d437a-f5da-11e7-a514-000c298fe627
Master_Info_File: /usr/local/mysql/data/master.info
SQL_Delay: 0
SQL_Remaining_Delay: NULL
Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
Master_Retry_Count: 86400
Master_Bind:
Last_IO_Error_Timestamp:
Last_SQL_Error_Timestamp:
Master_SSL_Crl:
Master_SSL_Crlpath:
Retrieved_Gtid_Set:
Executed_Gtid_Set:
Auto_Position: 0
Replicate_Rewrite_DB:
Channel_Name:
Master_TLS_Version:
1 row in set (0.00 sec)
```
> 检查Slave_IO_Running: Yes，Slave_SQL_Running: Yes证明搭建成功
