---
title: Mysql安装手册
tags:
  - mysql
categories:
  - db
date: 2019-01-26 02:00:00

---
>mysql 安装相关的内容

<!-- more -->

软件准备
--------

选择绿色安装。数据库版本是5.7.22。

主机系统环境
------------

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/91fdb214232ced7ed657aa473462337c.png)

创建用户和组
------------
```
$> groupadd mysql
$> useradd -r -g mysql mysql
```
数据库软件准备
--------------
```
$> tar -xzvf mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz
$> mv mysql-5.7.22-linux-glibc2.12-x86_64 /usr/local/mysql
$> chown mysql:mysql -R /usr/local/mysql
#数据目录创建，建议新挂一个磁盘到/opt下，然后在opt下创建data目录专门存储mysql数据
$>mkdir /opt/data
$>chown -R mysql:mysql /opt/data
```
设置PATH路径
------------
```
$>vi ~/.bash_profile
```
设置如下图：

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/2bb12b141f9ac28ff39b1248f537e1ff.png)

保存后source生效
```
$> source ~/.bash_profile
```
初始化数据库
------------
```
$> cd /usr/local/mysql/bin
$> ./mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/opt/data/
```
>记住初始密码（ZBy%#Uh45%gj），如下图
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/10648641329e44faf01eb00e182c6646.png)

配置数据库
----------
```
$>vi /etc/my.cnf
#配置如下
[mysqld]
basedir = /usr/local/mysql/
datadir = /opt/data/
character-set-server = utf8
pid-file=/opt/data/mysqld.pid
back_log=1024
wait_timeout=1800
max_connections=3200
max_user_connections=800
innodb_thread_concurrency=4
skip-name-resolve #在5.7.18安装是使用此参数会有问题，不能
innodb_buffer_pool_size=100M
innodb_log_buffer_size=20M
read_buffer_size=10M
sort_buffer_size=10M
read_rnd_buffer_size=10M
server-id=100
log-bin = master
[mysql]
default-character-set=utf8
```
配置解释
--------

1.  back_log=1024
>back_log的值指出在mysql暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。也就是说，如果mysql的连接数据达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源。

2.  wait_timeout=1800
>mysql连接闲置超过此设定值的时间后将会被强行关闭，单位是秒。

3.  max_connections=3200
>指mysql的最大连接数，如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，当然这是建立在机器能支撑的情况下，因为如果连接数越多，介于mysql会为每个连接提供连接缓冲区，就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。

4.  max_user_connections=800
>指每个数据库用户的最大连接。针对某一个账号的所有客户端并行连接到mysql服务的最大并行连接数。简单说是指同一个账号能够同时连接到mysql服务的最大连接数。设置为0表示不限制。

5.  innodb_thread_concurrency=CPU核数*2
>此值应设为CPU核数的2倍。

6.  skip-name-resolve
>禁止mysql对外部连接进行DNS解析，使用这一选项可以消除mysql进行DNS解析的时间。

7.  innodb_buffer_pool_size<=服务器内存*0.7
>用于缓存索引和数据的内存大小。

8.  innodb_log_buffer_size=20M
>InnoDB存储引擎的事务日志所使用的缓冲区。

9.  read_buffer_size=10M
>   这个值代表读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区。

10.  sort_buffer_size=10M
>执行排序使用的缓冲大小。

11.  read_rnd_buffer_size=10M
>随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，mysql会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。

12.  binlog=日志名字
>mysql的二进制日志是mysql最重要的日志，它记录了所有的DDL和DML语句，以事件形式记录，还包含语句所执行的消耗的时间，mysql的二进制日志是事务安全型的。

启动mysql
---------
```
$>cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
$>service mysqld start
```
修改密码
--------
```
$>mysql_secure_installation
```
>图中划红线的是修改密码部分，旧密码是初始化数据库时生成的随机密码。

![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/ora_mysql_install/c35978840d79c90476c7e0fbaafb3170.png)

centos7配置开机启动
---
配置文件创建
----------
```
[root@localhost ~]# touch /usr/lib/systemd/system/mysqld.service
[root@localhost ~]# cd /usr/lib/systemd/system
#编辑上一步创建的mysqld.service文件，加入如下内容:
[root@localhost system]# vi mysqld.service
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
```
>备注:ExecStart=/usr/local/mysql/bin/mysqld(此处请对应修改为mysql程序所在的路径)

启动mysql
-----------
```
[root@localhost system]# systemctl enable mysqld
[root@localhost system]# systemctl start mysqld
```
