---
title: postgresql绿色安装
tags:
  - postgresql
categories:
  - db
date: 2019-01-06 00:00:00
---
> postgresql绿色程序在linux系统中安装。
<!-- more -->
# 配置
```
export PGDATA=/home/postgres/pgdata
export PGUSER=postgres
export PGPORT=5432
export PATH=/home/postgres/pgsql/bin:$PATH
```

# 初始化数据目录

```
initdb -D /home/postgres/pgdata -E UTF8
```
# 配置
```
vi pgdata/postgresql.conf
llisten_addresses='*'

vi pgdata/pg_hba.conf
host	all	all	0.0.0.0/0	trust
```

# 启动
```
pg_ctl -D /home/postgres/pgdata -l logfile.log start
psql -d postgres
```