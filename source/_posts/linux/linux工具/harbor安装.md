---
title: harbor安装
date: 2019-09-30 01:00:00
tags: 
- linux 
categories: 
- tool
---


> harbor安装

<!-- more -->

## harbor安装

```
##先安装docker和docker-compose
[root@sdw3 ~]# wget https://github.com/vmware/harbor/releases/download/v1.2.0/harbor-offline-installer-v1.2.0.tgz
[root@sdw3 ~]# tar -zxf harbor-offline-installer-v1.2.0.tgz  -C /usr/local/
[root@sdw3 ~]# cd /usr/local/harbor/
[root@sdw3 ~]# vim /usr/local/harbor/harbor.cfg
hostname = rgs.unixfbi.com
#邮箱配置
email_server = smtp.qq.com
email_server_port = 25
email_username = unixfbi@unixfbi.com
email_password =12345678
email_from = UnixFBI <unixfbi@unixfbi.com>
email_ssl = false
#禁止用户注册
self_registration = off
#设置只有管理员可以创建项目
project_creation_restriction = adminonly
[root@sdw3 ~]# /usr/local/harbor/install.sh

```
## 启动compose

```
启动Harbor
# docker-compose start
停止Harbor
# docker-comose stop
重启Harbor
# docker-compose restart
```
## 测试
```
在浏览器输入rgs.unixfbi.com，因为我配置的域名为rgs.unixfbi.com。请大家根据自己的配置情况输入访问的域名；
默认账号密码： admin / Harbor12345 登录后修改密码
```
## 上传下载镜像

```

[root@sdw3 ~]# vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --insecure-registry rgs.unixfbi.com
[root@sdw3 ~]# systemctl  restart docker
[root@sdw3 ~]# docker login rgs.unixfbi.com
[root@sdw3 ~]# docker push rgs.unixfbi.com/library/centos7.1:0.1
创建Dockerfile
# vim Dockerfile 
FROM centos:centos7.1.1503
ENV TZ "Asia/Shanghai"
创建镜像
# docker build -t rgs.unixfbi.com/library/centos7.1:0.1 .
把镜像push到Harbor
# docker login rgs.unixfbi.com
# docker push rgs.unixfbi.com/library/centos7.1:0.1
如果不是自己创建的镜像，记得先执行 docker tags 给镜像做tag
例如：
# docker pull busybox
# docker tag busybox:latest rgs.unixfbi.com/library/busybox:latest
# docker push rgs.unixfbi.com/library/busybox:latest
```