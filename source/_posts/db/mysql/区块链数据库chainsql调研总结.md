---
title:  区块链数据库chainsql调研总结
tags:
- mysql chainsql 
categories: 
- db 
date: 2019-05-17 02:00:00
---
> 区块链数据库chainsql调研总结
<!-- more -->

## ChainSQL网络搭建

### 安装mysql并配置
  查阅mysql安装文档
```
CREATE DATABASE IF NOT EXISTS chainsql DEFAULT CHARSET utf8;
配置文件加入
character_set_server = utf8
max_connections=10000
```

### 验证结点搭建
#### 验证节点公私钥的生成
多个验证结点重复一下步骤
```
chainsqld_classic.exe validation_create
```
> 在0.30.3版本之前，执行这一命令要提前启动chainsqld进程，是因为下面的validation_create命令要向进程发送rpc请求，如果进程启动不成功，命令会返回错误。0.30.3及之后的版本可以不启动chainsqld程序直接返回结果。

#### 配置
为方便把所有的配置都写下面
节点1
```

############################################################################################
###
### 这是一个单点启动的配置文件（自己作为验证节点），只做测试使用，真正运行时不会使用单点验证
### 查看具体配置说明请看https://github.com/ChainSQL/chainsqld/blob/master/doc/chainsqld-example.cfg
### 及 https://github.com/ChainSQL/chainsqld/blob/master/doc/ChainSQLDesign.md
###
############################################################################################

#端口配置列表
[server]
port_rpc_admin_local
port_peer
port_ws_admin_local

#http端口配置
[port_rpc_admin_local]
port = 5006
ip = 0.0.0.0
admin = 127.0.0.1
protocol = http

#peer端口配置，用于p2p节点发现
[port_peer]
port = 5126
ip = 0.0.0.0
protocol = peer

#websocket端口配置
[port_ws_admin_local]
port = 6006
ip = 0.0.0.0
admin = 127.0.0.1
protocol = ws


#-------------------------------------------------------------------------------

[node_size]
medium

# This is primary persistent datastore for rippled.  This includes transaction
# metadata, account states, and ledger headers.  Helpful information can be
# found here: https://ripple.com/wiki/NodeBackEnd
# delete old ledgers while maintaining at least 2000. Do not require an
# external administrative command to initiate deletion.
# 区块数据存储配置，windows下用NuDB,Linux/Mac下用RocksDB
[node_db]
type=NuDB
path=./NuDB
open_files=2000
filter_bits=12
cache_mb=256
file_size_mb=8
file_size_mult=2

#是否全节点
[ledger_history]
full

#sqlite数据库（存储区块头数据，交易概要数据）
[database_path]
./db

# This needs to be an absolute directory reference, not a relative one.
# Modify this value as required.
[debug_logfile]
./debug.log

#时间服务器，用于不同节点单时间同步
[sntp_servers]
time.windows.com
time.apple.com
time.nist.gov
pool.ntp.org

# Where to find some other servers speaking the Ripple protocol.
# 2016-12-15 15:00:00
# 要连接的其它节点的Ip及端口
[ips]
127.0.0.1 5127
127.0.0.1 5128
127.0.0.1 5129

# Public keys of the validators that this rippled instance trusts.
# 信任节点列表（信任节点的公钥列表）
[validators]
n9LePgsBc7e3BNDdKESk9vppBD2fTNtwerfbqL5fB2MmkahirRNN
n9LDnydvRMR1sbGDyMHaSU7QHhVCesJtDjGDNxQv3chmXsBJYfY4
n9K8YTeGvm6u6As7JRBzomCtG7LNPxz9vmPpSrnCLnJEJ3u4nYHu

# 本节点私钥（如不配置，不参与共识）
[validation_seed]
xnKPgGfu1FkvVbBR8CXBpmVNEXvPi

# 本节点公钥
[validation_public_key]
n9KrVLNS8sBgaVLHZiciUeWUuKH9Q8RoLf2bNGRLdLJeuNn338yH

# Turn down default logging to save disk space in the long run.
#Valid values here are trace, debug, info, warning, error, and fatal
#日志级别，一般设置为warning级别
[rpc_startup]
{ "command": "log_level", "severity": "warning" }

# If ssl_verify is 1, certificates will be validated.
# To allow the use of self-signed certificates for development or internal use,
# set to ssl_verify to 0.
[ssl_verify]
0

#禁用某些支持但未不需要启用的特性,(开启会影响稳定性)
[veto_amendments]
42EEA5E28A97824821D4EF97081FE36A54E9593C6E4F20CBAE098C69D2E072DC fix1373
740352F2412A9909880C23A559FCECEDA3BE2126FED62FC7660D628A06927F11 Flow
E2E6F2866106419B88C50045ACE96368558C345566AC8F2BDF5A5B5587F0E6FA fix1368
C6970A8B603D8778783B61C0D445C23D1633CCFAEF0D43E7DBCD1521D34BD7C3 SHAMapV2
C1B8D934087225F509BEB5A8EC24447854713EE447D277F69545ABFA0E0FD490 Tickets
86E83A7D2ECE3AD5FA87AB2195AE015C950469ABF0B72EAACED318F74886AE90 CryptoConditionsSuite
1562511F573A19AE9BD103B5D6B9E01B3B46805AEC5D3C4805C902B514399146 CryptoConditions
3012E8230864E95A58C60FD61430D7E1B4D3353195F2981DC12B0C7C0950FFAC FlowCross


#chainsql数据库配置，根据自己的机子
[sync_db]
type=mysql
host=192.168.106.174
port=3306
user=root
pass=123456
db=chainsqla
first_storage=0
charset=utf8

# 开户自动同步后，节点运行情况下会去自动同步新建的表，开启这个开关，或者使用sync_tables标签的配置，否则无法同步表
[auto_sync]
1
```

节点2
```

############################################################################################
###
### 这是一个单点启动的配置文件（自己作为验证节点），只做测试使用，真正运行时不会使用单点验证
### 查看具体配置说明请看https://github.com/ChainSQL/chainsqld/blob/master/doc/chainsqld-example.cfg
### 及 https://github.com/ChainSQL/chainsqld/blob/master/doc/ChainSQLDesign.md
###
############################################################################################

#端口配置列表
[server]
port_rpc_admin_local
port_peer
port_ws_admin_local

#http端口配置
[port_rpc_admin_local]
port = 5007
ip = 0.0.0.0
admin = 127.0.0.1
protocol = http

#peer端口配置，用于p2p节点发现
[port_peer]
port = 5127
ip = 0.0.0.0
protocol = peer

#websocket端口配置
[port_ws_admin_local]
port = 6007
ip = 0.0.0.0
admin = 127.0.0.1
protocol = ws


#-------------------------------------------------------------------------------

[node_size]
medium

# This is primary persistent datastore for rippled.  This includes transaction
# metadata, account states, and ledger headers.  Helpful information can be
# found here: https://ripple.com/wiki/NodeBackEnd
# delete old ledgers while maintaining at least 2000. Do not require an
# external administrative command to initiate deletion.
# 区块数据存储配置，windows下用NuDB,Linux/Mac下用RocksDB
[node_db]
type=NuDB
path=./NuDB
open_files=2000
filter_bits=12
cache_mb=256
file_size_mb=8
file_size_mult=2

#是否全节点
[ledger_history]
full

#sqlite数据库（存储区块头数据，交易概要数据）
[database_path]
./db

# This needs to be an absolute directory reference, not a relative one.
# Modify this value as required.
[debug_logfile]
./debug.log

#时间服务器，用于不同节点单时间同步
[sntp_servers]
time.windows.com
time.apple.com
time.nist.gov
pool.ntp.org

# Where to find some other servers speaking the Ripple protocol.
# 2016-12-15 15:00:00
# 要连接的其它节点的Ip及端口
[ips]
127.0.0.1 5126
127.0.0.1 5128
127.0.0.1 5129

# Public keys of the validators that this rippled instance trusts.
# 信任节点列表（信任节点的公钥列表）
[validators]
n9KrVLNS8sBgaVLHZiciUeWUuKH9Q8RoLf2bNGRLdLJeuNn338yH
n9LDnydvRMR1sbGDyMHaSU7QHhVCesJtDjGDNxQv3chmXsBJYfY4
n9K8YTeGvm6u6As7JRBzomCtG7LNPxz9vmPpSrnCLnJEJ3u4nYHu

# 本节点私钥（如不配置，不参与共识）
[validation_seed]
xxEEippTRiy9MxaUU7yhKY2SXsCCr

# 本节点公钥
[validation_public_key]
n9LePgsBc7e3BNDdKESk9vppBD2fTNtwerfbqL5fB2MmkahirRNN

# Turn down default logging to save disk space in the long run.
#Valid values here are trace, debug, info, warning, error, and fatal
#日志级别，一般设置为warning级别
[rpc_startup]
{ "command": "log_level", "severity": "warning" }

# If ssl_verify is 1, certificates will be validated.
# To allow the use of self-signed certificates for development or internal use,
# set to ssl_verify to 0.
[ssl_verify]
0

#禁用某些支持但未不需要启用的特性,(开启会影响稳定性)
[veto_amendments]
42EEA5E28A97824821D4EF97081FE36A54E9593C6E4F20CBAE098C69D2E072DC fix1373
740352F2412A9909880C23A559FCECEDA3BE2126FED62FC7660D628A06927F11 Flow
E2E6F2866106419B88C50045ACE96368558C345566AC8F2BDF5A5B5587F0E6FA fix1368
C6970A8B603D8778783B61C0D445C23D1633CCFAEF0D43E7DBCD1521D34BD7C3 SHAMapV2
C1B8D934087225F509BEB5A8EC24447854713EE447D277F69545ABFA0E0FD490 Tickets
86E83A7D2ECE3AD5FA87AB2195AE015C950469ABF0B72EAACED318F74886AE90 CryptoConditionsSuite
1562511F573A19AE9BD103B5D6B9E01B3B46805AEC5D3C4805C902B514399146 CryptoConditions
3012E8230864E95A58C60FD61430D7E1B4D3353195F2981DC12B0C7C0950FFAC FlowCross


#chainsql数据库配置，根据自己的机子
[sync_db]
type=mysql
host=192.168.106.174
port=3306
user=root
pass=123456
db=chainsqlb
first_storage=0
charset=utf8

# 开户自动同步后，节点运行情况下会去自动同步新建的表，开启这个开关，或者使用sync_tables标签的配置，否则无法同步表
[auto_sync]
1
```
节点3
```

############################################################################################
###
### 这是一个单点启动的配置文件（自己作为验证节点），只做测试使用，真正运行时不会使用单点验证
### 查看具体配置说明请看https://github.com/ChainSQL/chainsqld/blob/master/doc/chainsqld-example.cfg
### 及 https://github.com/ChainSQL/chainsqld/blob/master/doc/ChainSQLDesign.md
###
############################################################################################

#端口配置列表
[server]
port_rpc_admin_local
port_peer
port_ws_admin_local

#http端口配置
[port_rpc_admin_local]
port = 5008
ip = 0.0.0.0
admin = 127.0.0.1
protocol = http

#peer端口配置，用于p2p节点发现
[port_peer]
port = 5128
ip = 0.0.0.0
protocol = peer

#websocket端口配置
[port_ws_admin_local]
port = 6008
ip = 0.0.0.0
admin = 127.0.0.1
protocol = ws


#-------------------------------------------------------------------------------

[node_size]
medium

# This is primary persistent datastore for rippled.  This includes transaction
# metadata, account states, and ledger headers.  Helpful information can be
# found here: https://ripple.com/wiki/NodeBackEnd
# delete old ledgers while maintaining at least 2000. Do not require an
# external administrative command to initiate deletion.
# 区块数据存储配置，windows下用NuDB,Linux/Mac下用RocksDB
[node_db]
type=NuDB
path=./NuDB
open_files=2000
filter_bits=12
cache_mb=256
file_size_mb=8
file_size_mult=2

#是否全节点
[ledger_history]
full

#sqlite数据库（存储区块头数据，交易概要数据）
[database_path]
./db

# This needs to be an absolute directory reference, not a relative one.
# Modify this value as required.
[debug_logfile]
./debug.log

#时间服务器，用于不同节点单时间同步
[sntp_servers]
time.windows.com
time.apple.com
time.nist.gov
pool.ntp.org

# Where to find some other servers speaking the Ripple protocol.
# 2016-12-15 15:00:00
# 要连接的其它节点的Ip及端口
[ips]
127.0.0.1 5126
127.0.0.1 5127
127.0.0.1 5129

# Public keys of the validators that this rippled instance trusts.
# 信任节点列表（信任节点的公钥列表）
[validators]
n9KrVLNS8sBgaVLHZiciUeWUuKH9Q8RoLf2bNGRLdLJeuNn338yH
n9LePgsBc7e3BNDdKESk9vppBD2fTNtwerfbqL5fB2MmkahirRNN
n9K8YTeGvm6u6As7JRBzomCtG7LNPxz9vmPpSrnCLnJEJ3u4nYHu

# 本节点私钥（如不配置，不参与共识）
[validation_seed]
xxbF6YeFdqYHVcqpErZZvFHayzytc

# 本节点公钥
[validation_public_key]
n9LDnydvRMR1sbGDyMHaSU7QHhVCesJtDjGDNxQv3chmXsBJYfY4

# Turn down default logging to save disk space in the long run.
#Valid values here are trace, debug, info, warning, error, and fatal
#日志级别，一般设置为warning级别
[rpc_startup]
{ "command": "log_level", "severity": "warning" }

# If ssl_verify is 1, certificates will be validated.
# To allow the use of self-signed certificates for development or internal use,
# set to ssl_verify to 0.
[ssl_verify]
0

#禁用某些支持但未不需要启用的特性,(开启会影响稳定性)
[veto_amendments]
42EEA5E28A97824821D4EF97081FE36A54E9593C6E4F20CBAE098C69D2E072DC fix1373
740352F2412A9909880C23A559FCECEDA3BE2126FED62FC7660D628A06927F11 Flow
E2E6F2866106419B88C50045ACE96368558C345566AC8F2BDF5A5B5587F0E6FA fix1368
C6970A8B603D8778783B61C0D445C23D1633CCFAEF0D43E7DBCD1521D34BD7C3 SHAMapV2
C1B8D934087225F509BEB5A8EC24447854713EE447D277F69545ABFA0E0FD490 Tickets
86E83A7D2ECE3AD5FA87AB2195AE015C950469ABF0B72EAACED318F74886AE90 CryptoConditionsSuite
1562511F573A19AE9BD103B5D6B9E01B3B46805AEC5D3C4805C902B514399146 CryptoConditions
3012E8230864E95A58C60FD61430D7E1B4D3353195F2981DC12B0C7C0950FFAC FlowCross


#chainsql数据库配置，根据自己的机子
[sync_db]
type=mysql
host=192.168.106.174
port=3306
user=root
pass=123456
db=chainsqlc
first_storage=0
charset=utf8

# 开户自动同步后，节点运行情况下会去自动同步新建的表，开启这个开关，或者使用sync_tables标签的配置，否则无法同步表
[auto_sync]
1
```
节点4
```

############################################################################################
###
### 这是一个单点启动的配置文件（自己作为验证节点），只做测试使用，真正运行时不会使用单点验证
### 查看具体配置说明请看https://github.com/ChainSQL/chainsqld/blob/master/doc/chainsqld-example.cfg
### 及 https://github.com/ChainSQL/chainsqld/blob/master/doc/ChainSQLDesign.md
###
############################################################################################

#端口配置列表
[server]
port_rpc_admin_local
port_peer
port_ws_admin_local

#http端口配置
[port_rpc_admin_local]
port = 5009
ip = 0.0.0.0
admin = 127.0.0.1
protocol = http

#peer端口配置，用于p2p节点发现
[port_peer]
port = 5129
ip = 0.0.0.0
protocol = peer

#websocket端口配置
[port_ws_admin_local]
port = 6009
ip = 0.0.0.0
admin = 127.0.0.1
protocol = ws


#-------------------------------------------------------------------------------

[node_size]
medium

# This is primary persistent datastore for rippled.  This includes transaction
# metadata, account states, and ledger headers.  Helpful information can be
# found here: https://ripple.com/wiki/NodeBackEnd
# delete old ledgers while maintaining at least 2000. Do not require an
# external administrative command to initiate deletion.
# 区块数据存储配置，windows下用NuDB,Linux/Mac下用RocksDB
[node_db]
type=NuDB
path=./NuDB
open_files=2000
filter_bits=12
cache_mb=256
file_size_mb=8
file_size_mult=2

#是否全节点
[ledger_history]
full

#sqlite数据库（存储区块头数据，交易概要数据）
[database_path]
./db

# This needs to be an absolute directory reference, not a relative one.
# Modify this value as required.
[debug_logfile]
./debug.log

#时间服务器，用于不同节点单时间同步
[sntp_servers]
time.windows.com
time.apple.com
time.nist.gov
pool.ntp.org

# Where to find some other servers speaking the Ripple protocol.
# 2016-12-15 15:00:00
# 要连接的其它节点的Ip及端口
[ips]
127.0.0.1 5126
127.0.0.1 5127
127.0.0.1 5128


# Public keys of the validators that this rippled instance trusts.
# 信任节点列表（信任节点的公钥列表）
[validators]
n9KrVLNS8sBgaVLHZiciUeWUuKH9Q8RoLf2bNGRLdLJeuNn338yH
n9LePgsBc7e3BNDdKESk9vppBD2fTNtwerfbqL5fB2MmkahirRNN
n9LDnydvRMR1sbGDyMHaSU7QHhVCesJtDjGDNxQv3chmXsBJYfY4


# 本节点私钥（如不配置，不参与共识）
[validation_seed]
xx98vyzwAXBkRijuxMAKeegh2CAa4

# 本节点公钥
[validation_public_key]
n9K8YTeGvm6u6As7JRBzomCtG7LNPxz9vmPpSrnCLnJEJ3u4nYHu

# Turn down default logging to save disk space in the long run.
#Valid values here are trace, debug, info, warning, error, and fatal
#日志级别，一般设置为warning级别
[rpc_startup]
{ "command": "log_level", "severity": "warning" }

# If ssl_verify is 1, certificates will be validated.
# To allow the use of self-signed certificates for development or internal use,
# set to ssl_verify to 0.
[ssl_verify]
0

#禁用某些支持但未不需要启用的特性,(开启会影响稳定性)
[veto_amendments]
42EEA5E28A97824821D4EF97081FE36A54E9593C6E4F20CBAE098C69D2E072DC fix1373
740352F2412A9909880C23A559FCECEDA3BE2126FED62FC7660D628A06927F11 Flow
E2E6F2866106419B88C50045ACE96368558C345566AC8F2BDF5A5B5587F0E6FA fix1368
C6970A8B603D8778783B61C0D445C23D1633CCFAEF0D43E7DBCD1521D34BD7C3 SHAMapV2
C1B8D934087225F509BEB5A8EC24447854713EE447D277F69545ABFA0E0FD490 Tickets
86E83A7D2ECE3AD5FA87AB2195AE015C950469ABF0B72EAACED318F74886AE90 CryptoConditionsSuite
1562511F573A19AE9BD103B5D6B9E01B3B46805AEC5D3C4805C902B514399146 CryptoConditions
3012E8230864E95A58C60FD61430D7E1B4D3353195F2981DC12B0C7C0950FFAC FlowCross


#chainsql数据库配置，根据自己的机子
[sync_db]
type=mysql
host=192.168.106.174
port=3306
user=root
pass=123456
db=chainsqld
first_storage=0
charset=utf8

# 开户自动同步后，节点运行情况下会去自动同步新建的表，开启这个开关，或者使用sync_tables标签的配置，否则无法同步表
[auto_sync]
1
```

### 启动
```
chainsqld_classic.exe
#如果想要加载之前的区块链数据启动，在某一全节点下执行下面的
chainsqld_classic.exe --load
```

### java操作


[ChainSql驱动][1]
[java例子][2]


  [1]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/code/libs.zip
  [2]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/code/chainsql.rar
