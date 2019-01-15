---
title: ADAM采集器用户手册
tags:
  - postgresql
categories:
  - db
date: 2019-01-14 00:00:00
---


## ADAM采集器简介
  采集器是ADAM整个产品的非常关键的部分，采集的信息的正确性、完整性、及时有效性，直接影响整个迁移过程。
<!-- more -->
## 主要步骤
1. 下载 ADAM 客户端
  - 登录阿里云,如果已有账号，可直接登录，否则请先注册后，在登录。
  - 进入到 数据库和应用服务 ADAM 界面，目前产品处于公测阶段，可以先申请公测资格，免费使用该产品。
  - 进入 ADAM 管理控制台， 在左侧导航栏中单击客户端下载。可以根据自己的运行环境，下载相应的客户端版本 Windows 64 位、 Linux 64 位，不要在32 位操作系统上运行以上版本。
2. 环境要求
  - 网络：需要可以连接到采集的数据库（通过tnsnames的方式进行采集）
  - 机器：
    * CPU: 2C+
    * 内存： 8g+
    * 磁盘空间： 20g+
    * 网路带宽： 千兆网+
3.  操作系统：
   - CentOS6.x+， RedHat6.x+， Oracle Linux6.x+等
   - Windows XP， 7， 8， 10+， Windows Server 2003+等
4.  数据库环境：
   - 开启归档
``` sql
SQL> archive log list
SQL> shutdown immediate
-- 启动数据库 mount 状态
SQL> startup mount
-- 打开归档
SQL> alter database archivelog;
-- 检查归档开启,Enabled 表示开启
SQL> archive log list;
-- 打开数据库
SQL> alter database open;
-- 切换归档
SQL> alter system switch logfile
```
  - 开启最小补充日志
``` sql
-- 检查最小补充日志是否开启,NO 表示未开启
SQL> select supplemental_log_data_min from v$database;
-- 开启最小补充日志
SQL> alter database add supplemental log data;
-- 验证已经开启,YES 表示已经开启
SQL> select supplemental_log_data_min from v$database;
```
  - 创建采集账号
```sql
--创建采集用户 eoa_user, 并设置密码为 eoaPASSW0RD
SQL> create user eoa_user identified by "eoaPASSW0RD" default tablespace users;
--为 eoa_user 用户赋予查询权限
SQL> grant connect,resource,select_catalog_role,select any dictionary to eoa_user;
--为 eoa_user 用户赋予执行 DBMS_LOGMNR 权限
-- (版本 10g 数据库需先执行: CREATE OR REPLACE PUBLIC SYNONYM dbms_logmnr FOR sys.dbms_logmnr)
SQL> grant execute on DBMS_LOGMNR to eoa_user;
--为 eoa_user 用户赋予执行 DBMS_METADATA 权限，查询数据对象 DDL 语句
SQL> grant execute on dbms_metadata to eoa_user;
-- 为 eoa_user 用户赋予查询事务权限
SQL> grant select any transaction to eoa_user;
```

## 采集数据
1. 解压采集包:
``` shell
$ tar -zxvf rainmeter-linux64.tar.gz
$ cd rainmeter
```
2. 采集方式：
  - 单次性采集
只采集一次全量数据，采集完进程就退出。
```
$ sh onekeyCollect.sh -h<ip> -u<username> -p<password> -d<service_name|sid>
$#sh onekeyCollect.sh -h192.168.106.174 -ueoa_user -p eoaPASSW0RD -dorcl -P1521
```
  - 周期性采集（最好采集1周以上）
     - 第一次采集全量数据，后面定时增量收集 SQL 相关数据直到执行取消命令（ctrl +c）。 或者直接进行周期性采集（包含1次全量采集）。
```
$ sh asynCollect.sh -h<ip> -u<username> -p<password> -d<service_name|sid> -P<port>
```

     - 采集结果打包
```
$ sh asynExport[.sh|.bat] -h<ip> -u<username> -p<password> -d<service_name|sid>
```
3. 清理采集账号
```
SQL> drop user eoa_user cascade;
```


## 上传采集数据包
1. 登录 阿里云 ADAM 控制台(https://rainbow-console.aliyun.com/studio.htm?spm=a2c4g.11186623.2.10.41c05d6egD44MV)，项目管理，新建项目.
2. 点击进入项目，上传数据包（数据包务必是 ZIP 压缩包）。点击分析。
