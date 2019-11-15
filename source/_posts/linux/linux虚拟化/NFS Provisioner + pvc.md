---
title:  NFS Provisioner + pvc
tags:
- 虚拟化
categories: 
- linux 
date: 2019-11-15 02:00:00
---
> NFS Provisioner + pvc
<!-- more -->

### NFS Provisioner + pvc
#### nfs server 

```
服务器：192.168.106.237
目录：  /export
```
#### 创建server account
```
[root@managementa ~]# cd fk
[root@managementa fk]# ls
bin  conf  dns-test  helm  ingress  kube-dashboard  kube-dns  kube-metrics-server  pki  PodSecurityPolicy.yaml  rancher  rancher.yaml
[root@managementa fk]# mkdir nfs-pvs
[root@managementa fk]# cd nfs-pvs/
[root@managementa nfs-pvs]# vi nfs-rbac.yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
  namespace: kube-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
  namespace: kube-system
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
  namespace: kube-system
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: kube-system      #替换成你要部署NFS Provisioner的 Namespace
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  namespace: kube-system
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  namespace: kube-system
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: kube-system      #替换成你要部署NFS Provisioner的 Namespace
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
[root@managementa nfs-pvs]# kubectl create -f nfs-rbac.yaml -n kube-system
serviceaccount/nfs-client-provisioner created
clusterrole.rbac.authorization.k8s.io/nfs-client-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/run-nfs-client-provisioner created
role.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner created
rolebinding.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner created
```


#### 部署 NFS Provisioner

```
[root@managementa nfs-pvs]# docker pull jmgao1983/nfs-client-provisioner
Using default tag: latest
latest: Pulling from jmgao1983/nfs-client-provisioner
605ce1bd3f31: Pull complete 
f7e4dcf21f30: Pull complete 
97ca8a640c74: Pull complete 
Digest: sha256:6e94e0ab8c447d8b393110790f8e76b2f6a769d57471d1ec1b9c4bda29359db9
Status: Downloaded newer image for jmgao1983/nfs-client-provisioner:latest
docker.io/jmgao1983/nfs-client-provisioner:latest
[root@managementa nfs-pvs]# docker tag docker.io/jmgao1983/nfs-client-provisioner:latest 192.168.106.117/library/nfs-client-provision:latest
[root@managementa nfs-pvs]# docker push 192.168.106.117/library/nfs-client-provision:latest
[root@managementa nfs-pvs]# docker rmi docker.io/jmgao1983/nfs-client-provisioner:latest
[root@managementa nfs-pvs]# vi nfs-provisioner-deploy.yaml 
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
  namespace: kube-system
  labels:
    app: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate      #---设置升级策略为删除再创建(默认为滚动更新)
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: 192.168.106.117/library/nfs-client-provision:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: nfs-client     #--- nfs-provisioner的名称，以后设置的storageclass要和这个保持一致
            - name: NFS_SERVER
              value: 192.168.106.237   #---NFS服务器地址，和 valumes 保持一致
            - name: NFS_PATH
              value: /export      #---NFS服务器目录，和 valumes 保持一致
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.106.237    #---NFS服务器地址
            path: /export         #---NFS服务器目录
[root@managementa nfs-pvs]# kubectl create -f nfs-provisioner-deploy.yaml  -n kube-system
[root@managementa nfs-pvs]# vi nfs-storage.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
  namespace: kube-system
  annotations:
    storageclass.kubernetes.io/is-default-class: "true" #---设置为默认的storageclass
provisioner: nfs-client    #---动态卷分配者名称，必须和上面创建的"provisioner"变量中设置的Name一致
parameters:
  archiveOnDelete: "true" 
[root@managementa nfs-pvs]# kubectl create -f nfs-storage.yaml 
```

#### 测试pvc

```
[root@managementa nfs-pvs]# vi test-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
spec:
  storageClassName: nfs-storage #---需要与上面创建的storageclass的名称一致
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Mi
[root@managementa nfs-pvs]# kubectl create -f test-pvc.yaml
[root@managementa nfs-pvs]# kubectl get pvc test-pvc
[root@managementa nfs-pvs]# cat test-pod.yaml 
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox:latest
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && exit 0 || exit 1"  #创建一个名称为"SUCCESS"的文件
    volumeMounts:
      - name: nfs-pvc
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: test-pvc
[root@managementa nfs-pvs]# kubectl create -f test-pod.yaml 
[root@managementa nfs-pvs]# mkdir test_yaml
[root@managementa nfs-pvs]# mv test-pod.yaml test_yaml/
[root@managementa nfs-pvs]# mv test-pvc.yaml test_yaml/
```