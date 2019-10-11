---
title: k8s 集群安装
date: 2019-10-10 02:00:00
tags: 
- 虚拟化 
categories: 
- linux
---


>k8s 集群安装

<!-- more -->

## 服务端安装 v1.16.0

```
[root@iz2zegdp0r569zzor0c3quz fk]# tar -xzvf kubernetes-server-linux-amd64.tar.gz 
##拷贝server bin下的文件到相应的目录

API-SERVER

[root@iz2zegdp0r569zzor0c3quz fk]# vi /usr/lib/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=etcd.service
Wants=etcd.service

[Service]
EnvironmentFile=/root/fk/conf/apiserver
ExecStart=/root/fk/bin/kube-apiserver $KUBE_API_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

[root@iz2zegdp0r569zzor0c3quz fk]# vi /root/fk/conf/apiserver
KUBE_API_ARGS="--etcd-servers=http://127.0.0.1:2379 --insecure-bind-address=0.0.0.0 --insecure-port=9090 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2"

KUBE_CONTROLLER_MANAGER

[root@iz2zegdp0r569zzor0c3quz fk]# vi /usr/lib/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=kube-apiserver.service
Wants=kube-apiserver.service

[Service]
EnvironmentFile=/root/fk/conf/controller-manager
ExecStart=/root/fk/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

[root@iz2zegdp0r569zzor0c3quz fk]# vi /root/fk/conf/controller-manager
KUBE_CONTROLLER_MANAGER_ARGS="--master=http://127.0.0.1:9090 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"


KUBE-SCHEDULER


[root@iz2zegdp0r569zzor0c3quz fk]# vi /usr/lib/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=kube-apiserver.service
Wants=kube-apiserver.service

[Service]
EnvironmentFile=/root/fk/conf/scheduler
ExecStart=/root/fk/bin/kube-scheduler $KUBE_SCHEDULER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

[root@iz2zegdp0r569zzor0c3quz fk]# vi /root/fk/conf/scheduler
KUBE_SCHEDULER_ARGS="--master=http://127.0.0.1:9090 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
```

## node端安装

```
KUBELET

##创建kubletconfig
kubectl config set-cluster kubernetes   --server=http://127.0.0.1:9090  --kubeconfig=bootstrap.kubeconfig
kubectl config set-context default  --cluster=kubernetes  --user=kubelet-bootstrap  --kubeconfig=bootstrap.kubeconfig
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig

[root@iz2zegdp0r569zzor0c3quz fk]# vi /usr/lib/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Wants=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
EnvironmentFile=/root/fk/conf/kubelet
ExecStart=/root/fk/bin/kubelet $KUBELET_ARGS
Restart=on-failure

[Install]
WantedBy=multi-user.target

[root@iz2zegdp0r569zzor0c3quz fk]# vi /root/fk/conf/kubelet
KUBELET_ARGS="--kubeconfig=/root/fk/conf/bootstrap.kubeconfig --hostname-override=172.17.9.158 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
[root@iz2zegdp0r569zzor0c3quz fk]# makdir /var/lib/kubelet



KUBE-PROXY
[root@iz2zegdp0r569zzor0c3quz fk]# vi /usr/lib/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.service
Wants=network.service

[Service]
EnvironmentFile=/root/fk/conf/proxy
ExecStart=/root/fk/bin/kube-proxy $KUBE_PROXY_ARGS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target

[root@iz2zegdp0r569zzor0c3quz fk]# vi /root/fk/conf/proxy
KUBE_PROXY_ARGS="--master=http://127.0.0.1:9090 --hostname-override=172.17.9.158 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"

```

## flannel组件

```
[root@iz2zegdp0r569zzor0c3quz log]# yum install flannel
[root@iz2zegdp0r569zzor0c3quz log]# vi /etc/sysconfig/flanneld
# Flanneld configuration options

# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD_ENDPOINTS="http://127.0.0.1:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_PREFIX="/etc/kubernetes/network"

# Any additional options that you want to pass
#FLANNEL_OPTIONS=""
[root@iz2zegdp0r569zzor0c3quz log]# etcdctl set /etc/kubernetes/network/config '{"Network": "172.20.0.0/16"}'


systemctl restart flanneld.service 




[root@iz2zegdp0r569zzor0c3quz log]# find / -name mk-docker-opts.sh
/usr/libexec/flannel/mk-docker-opts.sh
[root@iz2zegdp0r569zzor0c3quz log]# /usr/libexec/flannel/mk-docker-opts.sh -i
[root@iz2zegdp0r569zzor0c3quz log]# cat /run/flannel/subnet.env 
FLANNEL_NETWORK=172.20.0.0/16
FLANNEL_SUBNET=172.20.90.1/24
FLANNEL_MTU=1472
FLANNEL_IPMASQ=false
[root@iz2zegdp0r569zzor0c3quz log]# ifconfig docker0 172.20.90.1/24



systemctl restart docker

[root@iz2zegdp0r569zzor0c3quz log]# kubectl -s http://localhost:9090 get componentstatus
[root@iz2zegdp0r569zzor0c3quz log]# kubectl -s http://localhost:9090 get node
[root@iz2zegdp0r569zzor0c3quz log]# kubectl -s http://localhost:9090 cluster-info 
```

## controller

```

[root@iz2zegdp0r569zzor0c3quz fk]# cat redis-master-controller.yaml 
apiVersion: v1
kind: ReplicationController
metadata:
 name: redis-master
 labels:
  name: redis-master
spec:
 replicas: 1
 selector:
  name: redis-master
 template:
  metadata:
   labels:
    name: redis-master
  spec:
   containers:
   - name: master
     image: redis
     ports:
     - containerPort: 6379
[root@iz2zegdp0r569zzor0c3quz fk]# kubectl -s http://localhost:9090 create -f redis-master-controller.yaml 

解决
No API token found for service account "default", retry after the token is automatically created and added to the service account
openssl genrsa -out /root/fk/conf/serviceaccount.key 2048
vi /root/fk/conf/apiserver
加入
--service-account-key-file=/root/fk/conf/serviceaccount.key

vi /root/fk/conf/controller-manager
加入
--service-account-private-key-file=/root/fk/conf/serviceaccount.key

systemctl restart etcd kube-apiserver kube-controller-manager kube-scheduler












[root@iz2zegdp0r569zzor0c3quz fk]# kubectl -s http://localhost:9090 create -f hello-world-pod.yaml 
pod/hello-world created
[root@iz2zegdp0r569zzor0c3quz fk]# kubectl -s http://localhost:9090 get pod hello-world
NAME          READY   STATUS              RESTARTS   AGE
hello-world   0/1     ContainerCreating   0          25s


错误解决
failed pulling image "k8s.gcr.io/pause:3.1": Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

docker tag docker.io/mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
docker pull mirrorgooglecontainers/pause:3.1
```



