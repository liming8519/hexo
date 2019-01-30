
---
title: zabbix安装
tags:
  - tool
categories:
  - linux
date: 2019-01-30 01:00:00
---
> zabbix源码方式安装
<!-- more -->

## 环境准备
* [下载zabbix源码](https://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/4.0.3/zabbix-4.0.3.tar.gz/download)
* 上传zabbix源码到目标机器后执行下面命令
``` 
[root@master ~]# tar -zxvf zabbix-4.0.1.tar.gz
[root@master ~]# groupadd zabbix
[root@master ~]# useradd -g zabbix zabbix
```
* 配置数据库【安装mysql数据库略】
```
[root@master ~]# mysql -uroot -p<password>
MYSQL> create database zabbix character set utf8 collate utf8_bin;
MYSQL> grant all privileges on zabbix.* to 'zabbix'@'localhost' identified by 'zabbix';
MYSQL> quit;
[root@master ~]# cd zabbix-4.0.1/database/mysql/
[root@master mysql]# mysql -uzabbix -pzabbix zabbix < schema.sql
[root@master mysql]# #stop here if you are creating database for Zabbix proxy
[root@master mysql]# mysql -uzabbix -pzabbix zabbix < images.sql
[root@master mysql]# mysql -uzabbix -pzabbix zabbix < data.sql
```
## 安装server
```
[root@master ~]# ./configure --enable-server --enable-agent --with-mysql=/usr/local/mysql/bin/mysql_config --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2
[root@master ~]# make install
[root@master ~]# #启动服务：zabbix_server
[root@master ~]# su - zabbix
[zabbix@master ~]# zabbix_agentd
```
### 错误处理
* configure: error: LIBXML2 library not found 
```
[root@master ~]# yum -y install libxml2-devel
```
* configure: error: Invalid Net-SNMP directory - unable to find net-snmp-config
```
[root@master ~]# yum -y install net-snmp-devel
```
* configure: error: Unable to use libevent (libevent check failed)
```
[root@master ~]# yum install libevent-devel
```
* configure: error: Curl library not found 
```
[root@master ~]# yum -y install curl-devel
```
* zabbix_server: error while loading shared libraries: libmysqlclient.so.20:
 cannot open shared object file: No such file or
 directory
===========》
```
[root@master ~]# ln -s /usr/local/mysql/lib/libmysqlclient.so.20 /usr/lib64/mysql/libmysqlclient.so.20
[root@master ~]# ldconfig
```

## 安装配置web
```
[root@master ~]# yum install httpd
[root@master ~]# mkdir /var/www/html/zabbix
[root@master ~]# cd zabbix-4.0.1/frontends/php
[root@master php]# cp -a . /var/www/html/zabbix
[root@master php]# systemctl start httpd
[root@master php]# systemctl enable httpd
[root@master php]# yum install php php-mysql
```
打开URLhttp://ip/zabbix/setup.php
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/82372470ffa120dd2b39885d612a8c0c.png)
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/ef64285fe2b51d57b48e84d2cfd5831f.png)
按照提示处理依赖
```
[root@master ~]# yum install php-ldap
[root@master ~]# yum install php-gd
[root@master ~]# yum install php-xmlreader
[root@master ~]# yum install php-mbstring
[root@master ~]# yum install php-bcmath
#修改php.ini
date.timezone = Asia/Shanghai
post_max_size = 32M
max_execution_time = 300
max_input_time = 300
```
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/266ddc2c9f4edfcfdc418868cb6f0c2e.png)
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/eadf734873ebbb6ab31991418842d30a.png)
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/588b045598dc5f4117b9e37ea0be5bc6.png)
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/29e1fa333ef65708c055cfba81bdcf8c.png)
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/fe3f959eaed4507ab43f55b7f729a8f3.png)
cd /var/www/html/zabbix/conf
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/fe80eae3c35590b107a86a97a6e3f0b6.png)
## Zabbix-agent安装
```
[root@master ~]# tar -xzvf zabbix-4.0.1.tar.gz
[root@master zabbix-4.0.1]# cd zabbix-4.0.1/
[root@master zabbix-4.0.1]# ./configure --enable-agent
[root@master zabbix-4.0.1]# make install
[root@master zabbix-4.0.1]# vi /usr/local/etc/zabbix_agentd.conf #配置
[root@master zabbix-4.0.1]# groupadd zabbix
[root@master zabbix-4.0.1]# useradd -g zabbix zabbix
[root@master zabbix-4.0.1]# zabbix_agentd
```
## 选择中文语言
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/6d0cfe4bdaeba69974aa556bbb2d6ab5.png)
## Zabbix中文监控服务器图形图表显示乱码处理
复制下图中的字体到linux中的zabbix中
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/b228cea352906b5c030e113e8d39d20d.png)
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/7be29779af4f6ce40bba758a1bcb083c.png)
并做如下修改
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190130/bb2180500d7b1001bcb81854b9f9ec6e.png)

