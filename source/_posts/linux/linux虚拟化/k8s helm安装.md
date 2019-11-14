---
title:  k8s helm安装
tags:
- 虚拟化
categories: 
- linux 
date: 2019-11-14 02:00:00
---
> k8s helm安装
<!-- more -->

### k8s helm安装

#### helm安装
```
下载：https://github.com/helm/helm/releases
选择版本v2.15.0
[root@managementa fk]# mkdir helm
[root@managementa fk]# cd helm
[root@managementa helm]# wget https://get.helm.sh/helm-v2.15.0-linux-amd64.tar.gz
[root@managementa helm]# tar -xzvf helm-v2.15.0-linux-amd64.tar.gz 
[root@managementa helm]# cd linux-amd64/
[root@managementa linux-amd64]# ls
helm  LICENSE  README.md
[root@managementa linux-amd64]# mv helm ../../bin
[root@managementa linux-amd64]# cd ../../bin
[root@managementa bin]# pwd
/root/fk/bin
[root@managementa bin]# helm --kubeconfig /root/fk/conf/bootstrap.kubeconfig version
```

#### tiller安装
```
[root@managementa helm]# pwd
/root/fk/helm
[root@managementa helm]# vi rbac-config.yaml
2 files to edit
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
[root@managementa helm]#  kubectl create -f rbac-config.yaml
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created

上面可以用下面两句话代替
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
下面这句话要在tiller安装后执行，指定用户tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

[root@managementa ~]# docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.0
[root@managementa ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.0 192.168.106.117/library/tiller:v2.16.0
[root@managementa ~]# docker push 192.168.106.117/library/tiller:v2.16.0
[root@managementa ~]# docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.0

[root@managementa conf]# helm init --upgrade -i 192.168.106.117/library/tiller:v2.16.0 --kubeconfig /root/fk/conf/bootstrap.kubeconfig 
[root@managementa conf]# helm --kubeconfig /root/fk/conf/bootstrap.kubeconfig version
Client: &version.Version{SemVer:"v2.16.0", GitCommit:"e13bc94621d4ef666270cfbe734aaabf342a49bb", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.0", GitCommit:"e13bc94621d4ef666270cfbe734aaabf342a49bb", GitTreeState:"clean"}

[root@managementa test]# kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
deployment.apps/tiller-deploy patched
如果不执行上面语句那么会报错，如下：
Error: configmaps is forbidden: User "system:serviceaccount:kube-system:default" cannot list resource "configmaps" in API group "" in the namespace "kube-system"

注意上面下载的helm是2.15.0 , 经测试无论是下载helm2.15.0还是2.16.0上面的语句返回的Client都是2.16.0，server的版本跟镜像的tag相关。

```


#### tiller删除

```
[root@managementa conf]# helm reset --kubeconfig bootstrap.kubeconfig  -f
[root@managementa conf]# rm -rf /root/.helm
```

### 如何查看镜像的tag
```
docker search tiller
选择上面命令输出的内容在下面的网址里查询，就能查看到tag
https://hub.docker.com/r/library/
```

### helm教程

```
[root@managementa ~]# vi .bash_profile 
alias helm="helm --kubeconfig /root/fk/conf/bootstrap.kubeconfig"
[root@managementa ~]# source .bash_profile 
[root@managementa test]# helm ls
[root@managementa test]# helm repo list
NAME    URL                                             
stable  https://kubernetes-charts.storage.googleapis.com
local   http://127.0.0.1:8879/charts                    
[root@managementa test]# helm create hello-chart
Creating hello-chart
[root@managementa test]# helm lint ./hello-chart
==> Linting ./hello-chart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, no failures


helm search mysql
helm inspect stable/mysql
helm install stable/mysql
helm inspect values stable/mysql
helm install --name db-mysql --set mysqlRootPassword=anoyi  stable/mysql
```