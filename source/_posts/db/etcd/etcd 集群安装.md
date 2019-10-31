---
title: etcd 集群安装
date: 2019-10-10 02:00:00
tags: 
- etcd 
categories: 
- db
---


>etcd 集群安装

<!-- more -->

## ETCD v3.4.1下载

https://github.com/etcd-io/etcd/releases

## 单机安装
```
[root@iz2zegdp0r569zzor0c3quz ~]# tar zxf etcd-v3.4.1-linux-amd64.tar.gz
[root@iz2zegdp0r569zzor0c3quz ~]# mv etcd-v3.4.1-linux-amd64/etcd*  /usr/bin
##直接启动
[root@iz2zegdp0r569zzor0c3quz ~]# etcd
##服务启动
[vagrant@localhost etcd]$ vi /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=root
# set GOMAXPROCS to number of processors
ExecStart=/usr/bin/etcd
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target

vagrant@localhost etcd]$ vi /etc/etcd/etcd.conf

# [member]
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
#ETCD_SNAPSHOT_COUNT="10000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_LISTEN_PEER_URLS="http://localhost:2380"
ETCD_LISTEN_CLIENT_URLS="http://127.0.0.1:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
#ETCD_CORS=""
#
#[cluster]
#ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380"
# if you use different ETCD_NAME (e.g. test), set ETCD_INITIAL_CLUSTER value for this name,i.e. "test=http://..."
#ETCD_INITIAL_CLUSTER="default=http://localhost:2380"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://127.0.0.1:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_SRV=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#
#[proxy]
#ETCD_PROXY="off"
#ETCD_PROXY_FAILURE_WAIT="5000"
#ETCD_PROXY_REFRESH_INTERVAL="30000"
#ETCD_PROXY_DIAL_TIMEOUT="1000"
#ETCD_PROXY_WRITE_TIMEOUT="5000"
#ETCD_PROXY_READ_TIMEOUT="0"
#
#[security]
#ETCD_CERT_FILE=""
#ETCD_KEY_FILE=""
#ETCD_CLIENT_CERT_AUTH="false"
#ETCD_TRUSTED_CA_FILE=""
#ETCD_PEER_CERT_FILE=""
#ETCD_PEER_KEY_FILE=""
#ETCD_PEER_CLIENT_CERT_AUTH="false"
#ETCD_PEER_TRUSTED_CA_FILE=""
#
#[logging]
#ETCD_DEBUG="false"
# examples for -log-package-levels etcdserver=WARNING,security=DEBUG
#ETCD_LOG_PACKAGE_LEVELS=""
```

## 集群安装
```
[root@iz2zegdp0r569zzor0c3quz ~]# tar zxf etcd-v3.4.1-linux-amd64.tar.gz
[root@iz2zegdp0r569zzor0c3quz ~]# mv etcd-v3.4.1-linux-amd64/etcd*  /usr/bin
##直接启动
[root@iz2zegdp0r569zzor0c3quz ~]# etcd --name infra1 --listen-client-urls http://127.0.0.1:2379 --advertise-client-urls http://127.0.0.1:2379 --listen-peer-urls http://127.0.0.1:12380 --initial-advertise-peer-urls http://127.0.0.1:12380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof

[root@iz2zegdp0r569zzor0c3quz ~]#  etcd --name infra2 --listen-client-urls http://127.0.0.1:22379 --advertise-client-urls http://127.0.0.1:22379 --listen-peer-urls http://127.0.0.1:22380 --initial-advertise-peer-urls http://127.0.0.1:22380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof

[root@iz2zegdp0r569zzor0c3quz ~]#  etcd --name infra3 --listen-client-urls http://127.0.0.1:32379 --advertise-client-urls http://127.0.0.1:32379 --listen-peer-urls http://127.0.0.1:32380 --initial-advertise-peer-urls http://127.0.0.1:32380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof

##服务启动
[vagrant@localhost etcd]$ vi /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=root
# set GOMAXPROCS to number of processors
ExecStart=/usr/bin/etcd
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target

[vagrant@localhost etcd]$ vi /etc/etcd/etcd.conf

[member]

ETCD_NAME="k8s_master_ip_name" #范例：etcd1
ETCD_DATA_DIR="/work/etcd"
ETCD_LISTEN_PEER_URLS="http://k8s_master_ip:2380"
ETCD_LISTEN_CLIENT_URLS="http://127.0.0.1:2379,http://k8s_master_ip:2379"
[cluster]

ETCD_INITIAL_ADVERTISE_PEER_URLS="http://k8s_master_ip:2380"
ETCD_INITIAL_CLUSTER="k8s_master_ip_name=http://k8s_master_ip:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://k8s_master_ip:2379"

[vagrant@localhost etcd]$ etcdctl --write-out=table --endpoints=localhost:2379 member list
[vagrant@localhost etcd]$ etcdctl --write-out=table --endpoints=localhost:2379 enpoint status
```


