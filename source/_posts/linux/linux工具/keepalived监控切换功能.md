---
title: keepalived监控切换功能
tags:
  - tool
categories:
  - linux
date: 2019-02-13 03:00:00
---
> keepalived监控切换功能
<!-- more -->

## 配置本地yum源
```
cd /etc/yum.repos.d/
tar -cvf all.repo.tar *
rm -fr *.repo
vi cdiso.repo

[centos]
name=centos
baseurl=file:///root/c7
gpgcheck=0
enabled=1

#把iso挂载到/root/c7目录下
mkdir /root/c7
mount -o loop CentOS-7-x86_64-DVD-1611.iso /root/c7
```
## 基本环境
```
yum install keepalived
keepalived -v
systemctl enable keepalived
systemctl stop firewalld
systemctl disable firewalld

vi /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
```
## nginx监控脚本
```
cd /etc/keepalived/
vi check_nginx.sh
#!/bin/bash
counter=$(ps -C nginx --no-heading | wc -l)
if [ "${counter}" = "0" ]; then
    exit 1
else
    exit0
fi

chmod +x check_nginx.sh
```
## keepalived.conf 配置
```
cp keepalived.conf keepalived.conf.bak
vi keepalived.conf
##主节点的配置
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -5
    fall 1
    rise 1
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.24.70.251
    }
    track_script {
        chk_nginx
    }
}

##从节点的配置
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -5
    fall 3
    rise 2
}
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.24.70.251
    }
    track_script {
        chk_nginx
    }
}
```
## 启动keepalived
```
systemctl enable keepalived
systemctl start keepalived
```