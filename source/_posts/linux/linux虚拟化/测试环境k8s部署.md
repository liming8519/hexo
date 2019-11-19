---
title:  测试环境k8s部署
tags:
- 虚拟化
categories: 
- linux 
date: 2019-11-19 02:00:00
---
> 测试环境k8s部署
<!-- more -->

### 测试环境k8s部署

```
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
      - name: redis-master 
        image: 192.168.106.117/library/redis:3.2.9 
        ports: 
        - containerPort: 6379 
        volumeMounts:
        - name: redis-config-pvc
          mountPath: "/usr/local/etc/redis/"
        command: ["redis-server","/usr/local/etc/redis/redis-master.conf"]
      volumes:
      - name: redis-config-pvc
        persistentVolumeClaim:
          claimName: redis-config-pvc
--------
apiVersion: v1 
kind: Service 
metadata: 
  name: redis-master 
  labels: 
    name: redis-master 
spec: 
  ports: 
  - port: 6379 
    targetPort: 6379 
  selector: 
    name: redis-master
--------------------
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: busybox-test-dns
    role: master
  name: busybox
spec:
  containers:
  - name: busybox
    image: 192.168.106.117/library/busybox:latest
    command:
    - sleep
    - "3600"
	
	
	
	
	


---------------------

apiVersion: v1 
kind: ReplicationController 
metadata: 
  name: redis-slave 
  labels: 
    name: redis-slave 
spec: 
  replicas: 1 
  selector: 
    name: redis-slave 
  template: 
    metadata: 
      labels: 
        name: redis-slave 
    spec: 
      containers: 
      - name: redis-slave 
        image: 192.168.106.117/library/redis:3.2.9 
        command: ["redis-server","/usr/local/etc/redis/redis-slave.conf"]
        ports: 
        - containerPort: 6379 
        volumeMounts:
        - name: redis-config-pvc
          mountPath: "/usr/local/etc/redis/"
      volumes:
      - name: redis-config-pvc
        persistentVolumeClaim:
          claimName: redis-config-pvc
		
		
-------------------------------

apiVersion: v1 
kind: Service 
metadata: 
  name: redis-slave 
  labels: 
    name: redis-slave 
spec: 
  ports: 
  - port: 6379 
    targetPort: 6379 
  selector: 
    name: redis-slave
	
----------------------------------------------
# 创建redis-config-pvc存放redis config文件

#master
port 6379
bind 0.0.0.0
#daemonize yes
masterauth "123456"
requirepass "123456"
timeout 0
rdbcompression yes
dbfilename "redis.rdb"
dir "/data"
maxmemory 50gb
maxmemory-policy noeviction
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
notify-keyspace-events "gxE"


#slave
port 6379
bind 0.0.0.0
logfile "redis6500.log"
pidfile "6500.pid"
masterauth "123456"
requirepass "123456"
slave-read-only yes
slave-priority 100
timeout 0
rdbcompression yes
dbfilename "redis.rdb"
dir "/data"
maxmemory 50gb
maxmemory-policy noeviction
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
slaveof redis-master 6379


#sentinel
port 6379
protected-mode no
dir "/data"
sentinel myid bf64f29ff9578fb293d5d90ec85d083e2ae03725
sentinel monitor mymaster redis-master 6379 1
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 18000
sentinel auth-pass mymaster 123456
sentinel config-epoch mymaster 8735
sentinel leader-epoch mymaster 8735
sentinel known-slave mymaster redis-slave 6379
sentinel current-epoch 8735


----------------------------------

apiVersion: v1 
kind: ReplicationController 
metadata: 
  name: redis-sentinel 
  labels: 
    name: redis-sentinel 
spec: 
  replicas: 1 
  selector: 
    name: redis-sentinel 
  template: 
    metadata: 
      labels: 
        name: redis-sentinel 
    spec: 
      containers: 
      - name: redis-sentinel 
        image: 192.168.106.117/library/redis:3.2.9 
        command: ["redis-sentinel","/usr/local/etc/redis/sentinel.conf"]
        ports: 
        - containerPort: 6379 
        volumeMounts:
        - name: redis-config-pvc
          mountPath: "/usr/local/etc/redis/"
      volumes:
      - name: redis-config-pvc
        persistentVolumeClaim:
          claimName: redis-config-pvc
		
		
-------------------------------

apiVersion: v1 
kind: Service 
metadata: 
  name: redis-sentinel 
  labels: 
    name: redis-sentinel 
spec: 
  ports: 
  - port: 6379 
    targetPort: 6379 
  selector: 
    name: redis-sentinel
	
----------------------------------------------


创建tomcat 的 pvc



----------------------------------

apiVersion: v1 
kind: ReplicationController 
metadata: 
  name: tomcat-infoverisystem
  labels: 
    name: tomcat-infoverisystem
spec: 
  replicas: 1 
  selector: 
    name: tomcat-infoverisystem 
  template: 
    metadata: 
      labels: 
        name: tomcat-infoverisystem 
    spec: 
      containers: 
      - name: tomcat-infoverisystem
        image: 192.168.106.117/library/tomcat:8.5.35
        ports: 
        - containerPort: 8080 
        volumeMounts:
        - name: tomcat-war-pvc
          mountPath: "/usr/local/tomcat/webapps"
      volumes:
      - name: tomcat-war-pvc
        persistentVolumeClaim:
          claimName: tomcat-war-pvc
		
		
-------------------------------

apiVersion: v1 
kind: Service 
metadata: 
  name: tomcat-infoverisystem 
  labels: 
    name: tomcat-infoverisystem 
spec: 
  ports: 
  - port: 8080 
    targetPort: 8080 
  selector: 
    name: tomcat-infoverisystem
	
----------------------------------------------

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tomcat-infoverisystem
  annotations:
    kubernets.io/ingress.class: "nginx"
spec:
  rules:
  - http:
      paths:
      - path:
        backend:
          serviceName: tomcat-infoverisystem
          servicePort: 8080
```