---
title: FASTDFS安装步骤
tags:
  - tool
categories:
  - linux
date: 2019-03-04 05:00:00
---
> FASTDFS安装步骤
<!-- more -->

## FASTDFS安装步骤
```
0.init
[root@fileserver ~]# yum install gcc
[root@fileserver ~]# yum install unzip
1.libfastcommon
[root@fileserver ~]# unzip libfastcommon-1.0.36.zip
[root@fileserver ~]# cd libfastcommon-1.0.36
[root@localhost libfastcommon-1.0.36]# ./make.sh
[root@localhost libfastcommon-1.0.36]# ./make.sh install
2.FastDFS
[root@fileserver ~]# yum install perl
[root@fileserver ~]# unzip fastdfs-5.11.zip
[root@fileserver ~]# cd fastdfs-5.11
[root@fileserver fastdfs-5.11]# ./make.sh
[root@fileserver fastdfs-5.11]# ./make.sh install
3.3配置Tracker和Storage
[root@fileserver ~]# cd /etc/fdfs
[root@fileserver fdfs]# cp tracker.conf.sample tracker.conf
[root@fileserver fdfs]# mkdir /home/filedata
[root@fileserver fdfs]# mkdir /home/filedata/tracker
[root@fileserver fdfs]# vi tracker.conf

bind_addr=10.24.70.83
max_connections=500
base_path=/home/filedata/tracker

[root@fileserver fdfs]# cp storage.conf.sample storage.conf
[root@fileserver fdfs]# mkdir /home/filedata/storage
[root@fileserver fdfs]# mkdir /home/filedata/storage/info
[root@fileserver fdfs]# mkdir /home/filedata/storage/data
[root@fileserver fdfs]# vi storage.conf
bind_addr=10.24.70.83
base_path=/home/filedata/storage/info
store_path0=/home/filedata/storage/data
max_connections=500
tracker_server=10.24.70.83:22122
[root@fileserver etc]# chkconfig --add fdfs_trackerd
[root@fileserver etc]# chkconfig fdfs_trackerd on
[root@fileserver etc]# chkconfig fdfs_storaged on
[root@fileserver etc]# service fdfs_trackerd start
[root@fileserver etc]# service fdfs_storaged start
```

## 上传测试
```
#cd /etc/fdfs
#cp client.conf.sample client.conf
#vim client.conf
# Client 的数据和日志目录
base_path=/home/client
# Tracker端口
tracker_server=10.24.70.83:22122
#/usr/bin/fdfs_upload_file /etc/fdfs/client.conf namei.jpeg
```