---
title: MSSQL for linux 安装
tags:
  - mssql
categories:
  - db
date: 2019-01-011 00:00:00
---

> https://packages.microsoft.com/config/rhel/7/
> mssql地址
<!-- more -->
- 获取yum源

```
[root@yth-test bin]# curl https://packages.microsoft.com/config/rhel/7/mssql-server-2017.repo >/etc/yum.repos.d/mssql-server.repo

```

- 安装MSSQL

```
[root@yth-test bin]# yum install mssql-server
[root@yth-test bin]# cd /opt/mssql/bin
[root@yth-test bin]# ./mysql-conf setup

```

- 查看服务状态

```
systemctl status mssql
```

- 安装工具


```
[root@yth-test bin]# curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/prod.repo
[root@yth-test bin]# yum install mssql-tools
[root@yth-test bin]# pwd
/opt/mssql-tools/bin
[root@yth-test bin]# ./sqlcmd -S localhost -U sa

```
- win安装SSMS
> https://www.microsoft.com/zh-cn/download/details.aspx?id=8961