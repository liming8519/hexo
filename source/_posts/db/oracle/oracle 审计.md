---
title: oracle 审计
tags:
  - oracle
categories:
  - db
date: 2019-01-22 01:00:00

---



> ORACLE审计
<!-- more -->

## 查看审计参数

```
select value from v$parameter where name='audit_trail';
```

## 停止审计create session，并审计create table，truncate table ，alter table等

```
noaudit connect;
audit table by access whenever successful;
```

## 审计对象

```
audit delete on table_name;
```

## 查看语句审计

```
select * from sys.dba_stmt_audit_opts;
```

## 查看对象审计

```
select * from dba_obj_audit_opts;
```

## 查看审计信息

```
select * from DBA_AUDIT_OBJECT;
select * from dba_audit_statement;
select * from DBA_AUDIT_TRAIL;
```
