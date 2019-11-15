---
title:  k8s扩展插件
tags:
- 虚拟化
categories: 
- linux 
date: 2019-11-07 02:00:00
---
> k8s扩展插件
<!-- more -->

### flannel在docker中安装
#### flannel结点步骤

```
登录192.168.106.117 ,其他结点类似。
https://github.com/coreos/flannel/releases地址
[root@managementa ~] wget https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz
此版本的flanneld只支持etcd的v2版本的api，在此注意。
[root@managementa ~]# tar -xzvf flannel-v0.11.0-linux-amd64.tar.gz 
[root@managementa ~]# mv flanneld /usr/local/docker/
[root@managementa ~]# rm -fr README.md 
[root@managementa ~]# mv mk-docker-opts.sh /usr/local/docker/
[root@managementa ~]# rm -fr flannel-v0.11.0-linux-amd64.tar.gz
[root@managementa ~]# vim /etc/sysconfig/flanneld
FLANNEL_ETCD="http://192.168.106.237:2379"
FLANNEL_ETCD_KEY="/flannel/network"
FLANNEL_OPTIONS="-ip-masq=true -v=0"
FLANNEL_IFACE="192.168.106.117" #网卡 eth0 或IP
-ip-masq=true 这个参数的目的是让flannel进行ip伪装，而不让docker进行ip伪装。这么做的原因是如果docker进行ip伪装，流量再从flannel出去，其他host上看到的source ip就是flannel的网关ip，而不是docker容器的ip
-v 日志级别
[root@managementa ~]# vim /usr/lib/systemd/system/flanneld.service 
[Unit]
Description=Flanneld server
After=network.target
After=network-online.target
Wants=network-online.target
Before=docker.service
 
[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/flanneld
ExecStart=/usr/local/docker/flanneld -etcd-endpoints=${FLANNEL_ETCD} -iface=${FLANNEL_IFACE} -etcd-prefix=${FLANNEL_ETCD_KEY} $FLANNEL_OPTIONS
Restart=on-failure
 
[Install]
WantedBy=multi-user.target

```

#### flannel对应etcd配置

```
登录192.168.106.237
#向etcd 写入网段配置
[root@managementa ~] export ETCDCTL_API=2
[root@managementa ~] etcdctl --endpoint http://192.168.106.237:2379 mk /flannel/network/config '{"Network": "172.17.0.1/16","SubnetMin": "172.17.0.0", "SubnetMax": "172.17.254.0","Backend":{"Type":"vxlan"}}' 
{"Network": "172.17.0.1/16","SubnetMin": "172.17.0.0", "SubnetMax": "172.17.254.0","Backend":{"Type":"vxlan"}}
[root@sdw3 ~]# etcdctl --endpoints=http://192.168.106.237:2379 ls
/flannel
[root@sdw3 ~]# etcdctl --endpoints=http://192.168.106.237:2379 ls /flannel
/flannel/network
[root@sdw3 ~]# etcdctl --endpoints=http://192.168.106.237:2379 ls /flannel/network
/flannel/network/config
/flannel/network/subnets
[root@sdw3 ~]# etcdctl --endpoints=http://192.168.106.237:2379 ls /flannel/network/config
/flannel/network/config
[root@sdw3 ~]# etcdctl --endpoints=http://192.168.106.237:2379 get /flannel/network/config
{"Network": "172.17.0.1/16","SubnetMin": "172.17.0.0", "SubnetMax": "172.17.254.0","Backend":{"Type":"vxlan"}}
[root@sdw3 ~]# etcdctl --endpoints=http://192.168.106.237:2379 ls /flannel/network/subnets 
/flannel/network/subnets/172.17.63.0-24


错误解决
 Couldn't fetch network config: client: response is invalid json. The endpoint is probably not valid etcd cluster endpoint.
Nov 11 10:53:42 managementa flanneld: timed out
flannel的0.11版本临时不支持etcd3.4，并且不支持3版本的api。 下载etcd 3.3.17
https://github.com/etcd-io/etcd/releases/download/v3.3.17/etcd-v3.3.17-linux-amd64.tar.gz
```
#### 结点其他配置

```
[root@managementa ~] iptables -F
[root@managementa ~] iptables -X
[root@managementa ~] iptables -t nat -F
[root@managementa ~] iptables -t nat -X
[root@managementa ~] iptables -t mangle -F
[root@managementa ~] iptables -t mangle -X
[root@managementa ~] iptables -t raw -F
[root@managementa ~] iptables -t raw -X
[root@managementa ~] iptables -t security -F
[root@managementa ~] iptables -t security -X
[root@managementa ~] iptables -P INPUT ACCEPT
[root@managementa ~] iptables -P FORWARD ACCEPT
[root@managementa ~] iptables -P OUTPUT ACCEPT
**只刷新nat表**

[root@managementa ~]# systemctl daemon-reload
[root@managementa ~]# systemctl stop docker
[root@managementa ~]# systemctl stop flanneld
[root@managementa ~]# systemctl enable flanneld
Created symlink from /etc/systemd/system/multi-user.target.wants/flanneld.service to /usr/lib/systemd/system/flanneld.service.
[root@managementa ~]# systemctl start flanneld
[root@managementa ~]# mk-docker-opts.sh 
[root@managementa ~]# cat /run/docker
docker/          docker_opts.env  
[root@managementa ~]# cat /run/docker_opts.env 
DOCKER_OPT_BIP="--bip=172.17.63.1/24"
DOCKER_OPT_IPMASQ="--ip-masq=false"
DOCKER_OPT_MTU="--mtu=1450"
DOCKER_OPTS=" --bip=172.17.63.1/24 --ip-masq=false --mtu=1450"
按照上面的配置修改，如下：
[root@managementa ~]# vi /etc/docker/daemon.json 
{
    "registry-mirror":["http://192.168.106.117"],
    "insecure-registries":["192.168.106.117"],
    "bip":"172.17.63.1/24",
    "ip-masq":false,
    "mtu":1450

}
[root@managementa ~]# curl http://192.168.106.237:2379/v2/keys/flannel/network/subnets | python -m json.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   383  100   383    0     0   333k      0 --:--:-- --:--:-- --:--:--  374k
{
    "action": "get",
    "node": {
        "createdIndex": 9,
        "dir": true,
        "key": "/flannel/network/subnets",
        "modifiedIndex": 9,
        "nodes": [
            {
                "createdIndex": 10,
                "expiration": "2019-11-12T06:21:24.956806947Z",
                "key": "/flannel/network/subnets/172.17.63.0-24",
                "modifiedIndex": 10,
                "ttl": 85737,
                "value": "{\"PublicIP\":\"192.168.106.117\",\"BackendType\":\"vxlan\",\"BackendData\":{\"VtepMAC\":\"5e:8e:c4:02:e6:c8\"}}"
            }
        ]
    }
}
[root@managementa ~]# systemctl restart docker
~      
```
### k8s ca证书部署

```
下载 cfssl 工具：https://pkg.cfssl.org/
[root@managementa conf]# curl -s -L -o /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
[root@managementa conf]# curl -s -L -o /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
[root@managementa conf]# curl -s -L -o /bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
[root@managementa conf]# chmod +x /bin/cfssl*
[root@managementa conf]# mkdir /root/ssl
[root@managementa conf]# cd /root/ssl
[root@managementa ssl]# cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF

[root@managementa ssl]# cat > ca-csr.json << EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
[root@managementa ssl]# cfssl gencert -initca ca-csr.json | cfssljson -bare ca
2019/11/13 15:24:05 [INFO] generating a new CA key and certificate from CSR
2019/11/13 15:24:05 [INFO] generate received request
2019/11/13 15:24:05 [INFO] received CSR
2019/11/13 15:24:05 [INFO] generating key: rsa-2048
2019/11/13 15:24:05 [INFO] encoded CSR
2019/11/13 15:24:05 [INFO] signed certificate with serial number 714375892951147589454567376836425223244252998471
[root@managementa ssl]# ls ca*
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem



[root@managementa ssl]# cat > kubernetes-csr.json << EOF
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "192.168.106.117",
      "192.168.106.237",
      "192.168.106.118",
      "192.168.106.119",
      "192.168.106.128",
      "192.168.106.129",
      "192.168.106.103",
      "169.169.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

[root@managementa ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
[root@managementa ssl]# ls kubernetes*
kubernetes.csr  kubernetes-csr.json  kubernetes-key.pem  kubernetes.pem
验证证书
[root@managementa ssl]# openssl x509  -noout -text -in  kubernetes.pem
[root@managementa ssl]# cp *.pem ../fk/pki/
[root@managementa ssl]# cd ../fk/pki/
[root@managementa pki]# ls
ca.crt  ca-key.pem  ca.pem  kubernetes-key.pem  kubernetes.pem  server.crt  server.key
[root@managementa pki]# cd ../conf/
[root@managementa conf]# systemctl stop kube-controller-manager
[root@managementa conf]# cat controller-manager 
KUBE_CONTROLLER_MANAGER_ARGS="--master=http://127.0.0.1:9090 --logtostderr=false --log-dir=/var/log/kubernetes --v=2 --service-account-private-key-file=/root/fk/pki/kubernetes-key.pem --root-ca-file=/root/fk/pki/ca.pem" 
[root@managementa conf]# cat apiserver 
KUBE_API_ARGS="--enable-aggregator-routing --authorization-mode=RBAC --etcd-servers=http://192.168.106.237:2379 --insecure-bind-address=0.0.0.0 --insecure-port=9090 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2 --client-ca-file=/root/fk/pki/ca.pem --tls-cert-file=/root/fk/pki/kubernetes.pem --tls-private-key-file=/root/fk/pki/kubernetes-key.pem"
[root@managementa conf]# kubectl get secrets --all-namespaces
NAMESPACE         NAME                  TYPE                                  DATA   AGE
default           default-token-crcnd   kubernetes.io/service-account-token   3      2d
kube-node-lease   default-token-lh2nx   kubernetes.io/service-account-token   3      77m
kube-public       default-token-b76v6   kubernetes.io/service-account-token   3      2d
kube-system       default-token-jv9zg   kubernetes.io/service-account-token   3      2d
[root@managementa conf]# kubectl delete secrets default-token-crcnd
secret "default-token-crcnd" deleted
[root@managementa conf]# kubectl delete secrets default-token-lh2nx -n kube-node-lease
secret "default-token-lh2nx" deleted
[root@managementa conf]# kubectl delete secrets default-token-b76v6 -n kube-public
secret "default-token-b76v6" deleted
[root@managementa conf]#  kubectl delete secrets default-token-jv9zg -n kube-system
secret "default-token-jv9zg" deleted
[root@managementa conf]# kubectl get secrets --all-namespaces
No resources found
[root@managementa conf]# kubectl get serviceaccounts --all-namespaces
NAMESPACE         NAME      SECRETS   AGE
default           default   1         2d
kube-node-lease   default   1         78m
kube-public       default   1         2d
kube-system       default   1         2d
[root@managementa conf]# systemctl stop kube-apiserver
[root@managementa conf]# systemctl start kube-apiserver
[root@managementa conf]# systemctl start kube-controller-manager
[root@managementa fk]# cd kube-metrics-server/
[root@managementa kube-metrics-server]# ls
aggregated-metrics-reader.yaml  auth-delegator.yaml  auth-reader.yaml  metrics-apiservice.yaml  metrics-server-deployment.yaml  metrics-server-service.yaml  resource-reader.yaml
[root@managementa kube-metrics-server]# kubectl apply -f .
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
[root@managementa kube-metrics-server]# kubectl get pods --all-namespaces
NAMESPACE     NAME                              READY   STATUS    RESTARTS   AGE
kube-system   metrics-server-6f66574f8d-bx547   1/1     Running   0          9s
[root@managementa kube-metrics-server]# kubectl get pods --all-namespaces
NAMESPACE     NAME                              READY   STATUS    RESTARTS   AGE
kube-system   metrics-server-6f66574f8d-bx547   1/1     Running   0          11s
证书备份，只罗列内容：
[root@managementa pki]# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  kubernetes.csr  kubernetes-csr.json  kubernetes-key.pem  kubernetes.pem
[root@managementa pki]# cat ca-key.pem 
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA7FO8JNTypRkTj3eekQhXhikBGEu5ev6TW8KI700MuBaK0RmX
5vibkJ4vuUuvrL0QPoTPnlLs7f8XytMqtaEyQUsrv0R2BvC3Y+HckqNNaqtWCCUL
5KLs2c0FXeTvuX3PIx76Z0Si7Y5NnNusmkUc7zoQRXCwm13WtXTlj6RFImvpxehy
lsxOMaDjaT+U8I/hvvCaTC0rkB1eFyXH7Fw6rv2oui5rm67MBurjHyXjGMHX1WIE
JzF/NH5KKq0tD4bUyfQwLA8M9F9MvSHRvl7nE0moGsAY23r5D778sX2EzMTXHUNk
cO4bE9rqirI2UrBNjNlQHOBZ+Gu7hQMNYJdDPQIDAQABAoIBACfYrn5fUVI4+i1U
c+3sRCWgwEiCbBGq3tm34TLIAP8A/gLnl88f18r4gP9zHXm4nwaLih4dyUkPm8lc
9XSOa1TLAeNL/cKJz8INkQ1Ab4suvGC/LlQsjFbk1KTSNwFFjylSzdGfpwD632c1
OtMAGDLVzWyH5Z8soUkTHqmrfuSgjmYNsGOCM8PK2eF1knxv698mPJY0XU3X5yG2
e1Jl77MT8cJDbO+XuqBvmOWYJ1S55lsTRG58gHHBmhk6il/7i39SgGkHSFavNu5A
lCpkms9id0W5QZ7NethowgMeCTZZent4uMrkCJcm463/kHsGkT3Y9ea+iUNckHOS
DZ5CgEECgYEA9szT1N42r/kACo2NCWE3MEEDp80xTO9YBJs/6v1C/Z0iBbLDDvvW
HejljffUgh9nz0+OG5h9I31UKI6ojRxgIyST6yG2koWJg8EtUGwNqodixoeni07h
Ql04Tcy2QBTeurMafBouafTcKTVEHHp5TPCAzwz59z6jbv3RDfHZBVUCgYEA9SL3
Fy7o8ebs+5oldPLFhtbLFSTu6ELGn9REdG2z2wxOV68ICdjGyjE+njIupw7qS4AX
Smwd+uqApE0AmlTHz+3MJLWnE6m8rjcLG1zFapqDnd//3yS9E0bmQx8ov9IzeI18
QgF5OoXS/vBAlsvF2LTx4NfTHDTaDKzyTmFBxkkCgYBZUFTUszotQro+F23T2Cel
wdF1128g/Xjn6dseylqE92mJkGDAumiJWHBCiU6RbJYf0xWFbRDUWBWtu7rJnlw4
O5OAQyoUKllSogUpFoF3lhkr6Ym7g2dHof6vQQcvd54HCKvr/3mOhLtr+kfU2omt
S1gCFhsb28I/d4FBP6WfJQKBgQDcOa5cQIN8FyceHmy6NQRpz/wgoc6UELGakztw
kcG778FOGuwQ1JQ6v6TuwEyTPt4UOB40eQ8yBYzOjnMVM0dTMOJutFdGXf4pUUAE
NAMTc378zWl1Ee9fKxnggVS9h90/13QjZGmBvwpAiJyuHKFAv8ZxZdO+Cmk+a/0/
lzZdKQKBgCSLHELINvO+uUOXkoUXYYkrYJoH9uUJRFi8Gy5hdOpnMFYGcBiSbU2v
JUACtXBXN1ytmU5nYslKZpCfvzuD0xsfi//z/dDZDYDx8tbEldDJi/svPZu352Yn
rVnz661/xhUj5W3qCnlKE6kyleSStzZsVTTmtSdWXJUAw7XFEGFm
-----END RSA PRIVATE KEY-----
[root@managementa pki]# cat ca.pem
-----BEGIN CERTIFICATE-----
MIIDvjCCAqagAwIBAgIUfSG5ElG6kCDKiUxrkpEPDYyAa0cwDQYJKoZIhvcNAQEL
BQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0Jl
aUppbmcxDDAKBgNVBAoTA2s4czEPMA0GA1UECxMGU3lzdGVtMRMwEQYDVQQDEwpr
dWJlcm5ldGVzMB4XDTE5MTExMzA3MTkwMFoXDTI0MTExMTA3MTkwMFowZTELMAkG
A1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0JlaUppbmcxDDAK
BgNVBAoTA2s4czEPMA0GA1UECxMGU3lzdGVtMRMwEQYDVQQDEwprdWJlcm5ldGVz
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA7FO8JNTypRkTj3eekQhX
hikBGEu5ev6TW8KI700MuBaK0RmX5vibkJ4vuUuvrL0QPoTPnlLs7f8XytMqtaEy
QUsrv0R2BvC3Y+HckqNNaqtWCCUL5KLs2c0FXeTvuX3PIx76Z0Si7Y5NnNusmkUc
7zoQRXCwm13WtXTlj6RFImvpxehylsxOMaDjaT+U8I/hvvCaTC0rkB1eFyXH7Fw6
rv2oui5rm67MBurjHyXjGMHX1WIEJzF/NH5KKq0tD4bUyfQwLA8M9F9MvSHRvl7n
E0moGsAY23r5D778sX2EzMTXHUNkcO4bE9rqirI2UrBNjNlQHOBZ+Gu7hQMNYJdD
PQIDAQABo2YwZDAOBgNVHQ8BAf8EBAMCAQYwEgYDVR0TAQH/BAgwBgEB/wIBAjAd
BgNVHQ4EFgQUwDWSzTl7CZ7y4bMAGSeLLrfUmzYwHwYDVR0jBBgwFoAUwDWSzTl7
CZ7y4bMAGSeLLrfUmzYwDQYJKoZIhvcNAQELBQADggEBAHeAY4J5N+s4CKO1eR2/
e7MyPgbqDQ/pGB7LImf9cgfxd/Ufgr4RODXXDb+cHOYr+whzMfFWszzACqFTvjBj
0ZAUIAVOpBvBhNYFlrbEyupRJewb9cxAhXSLLGPr3EyzNtOOGdV6c6wih99sng+Z
I0SjwutkUpap5QfUo+/Up3QmpPixqUPa99U6Pi2u9Kr2qOk8pVv2uAHpF3Bu4Roi
jp04jD0FKLDUGuwf9QpMVW4BGvxmwqIXzTnY65ksz4ckdKamB3pMXxJ6iCRArnOn
anAbX024gNoTPcAbuP0TSNJjo4p1vg3agNQBIU88+fPQUzx+9tOkTiBtziwyhzcu
A7g=
-----END CERTIFICATE-----
[root@managementa pki]# cat ca.csr
-----BEGIN CERTIFICATE REQUEST-----
MIICqjCCAZICAQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAO
BgNVBAcTB0JlaUppbmcxDDAKBgNVBAoTA2s4czEPMA0GA1UECxMGU3lzdGVtMRMw
EQYDVQQDEwprdWJlcm5ldGVzMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
AQEA7FO8JNTypRkTj3eekQhXhikBGEu5ev6TW8KI700MuBaK0RmX5vibkJ4vuUuv
rL0QPoTPnlLs7f8XytMqtaEyQUsrv0R2BvC3Y+HckqNNaqtWCCUL5KLs2c0FXeTv
uX3PIx76Z0Si7Y5NnNusmkUc7zoQRXCwm13WtXTlj6RFImvpxehylsxOMaDjaT+U
8I/hvvCaTC0rkB1eFyXH7Fw6rv2oui5rm67MBurjHyXjGMHX1WIEJzF/NH5KKq0t
D4bUyfQwLA8M9F9MvSHRvl7nE0moGsAY23r5D778sX2EzMTXHUNkcO4bE9rqirI2
UrBNjNlQHOBZ+Gu7hQMNYJdDPQIDAQABoAAwDQYJKoZIhvcNAQELBQADggEBAHJQ
QJlDBiXnuZ7H/yTBSRu6LNNP1uuTriO514lS3DBt8NydHYz8B8XMufgzdDdHWWz9
sJXL5iw4MYOFXc9QhCBDLxFqNdCduMvn+cdVkAAAA1uk0G7R4VADV+w1ikB5qCFV
EgjCQsBajITkfqLDxarwZWc/nlRqsiXulWVRjKTmaXGULg0PSslZECXzxb19tvp4
qPEBlqZDXnfVNB0KSL5lJvbiyqeJnIdveOy+qer5aYvjtPbpk9quE2ufTuQCsBGx
FyZasR7Dk6s4dpW3cjRqsTHryy2xeZn3fM4uGDzriyHySW2hOAErMW3dHngndM7E
SqXAYR3oJKSjnGskp0E=
-----END CERTIFICATE REQUEST-----
[root@managementa pki]# cat ca-csr.json
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
[root@managementa pki]# cat ca-config.json
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
[root@managementa pki]# cat kubernetes-csr.json
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "192.168.106.117",
      "192.168.106.237",
      "192.168.106.118",
      "192.168.106.119",
      "192.168.106.128",
      "192.168.106.129",
      "192.168.106.103",
      "169.169.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
[root@managementa pki]# cat kubernetes.csr
-----BEGIN CERTIFICATE REQUEST-----
MIIDgTCCAmkCAQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAO
BgNVBAcTB0JlaUppbmcxDDAKBgNVBAoTA2s4czEPMA0GA1UECxMGU3lzdGVtMRMw
EQYDVQQDEwprdWJlcm5ldGVzMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
AQEA2Yhv0Je+Yxs9rqGkIBd3Jy1loq9bnHMgA8nVpu//QVR4n8Fqq3Lvt3/XnJg3
6u/l0UufVQD/hd+MWKL/bqnJMYb5YGBZCd1P5VOE3YyKmIgiea79y1XUjl5Bvn8i
5ReQINT3RIrXqdw0Qm/bSSqvopTC6VPSN8Ax6eI0Lnr1sLdTH6zA1UsUtKCHmM0W
h92PF158dihjy/YrtjMFe91P9dOJocbwwARzxJlRO21OSJC4P/zkpDWBtd8Oxjs5
UIkjXI6QDxx/cStGP2lJAm39Wgu2gg835gglhD8oeVnwfbRl52BBdKmH2UL/Nb/o
VkUVHK1W6bosFqE3SLDHmhPc9QIDAQABoIHWMIHTBgkqhkiG9w0BCQ4xgcUwgcIw
gb8GA1UdEQSBtzCBtIIKa3ViZXJuZXRlc4ISa3ViZXJuZXRlcy5kZWZhdWx0ghZr
dWJlcm5ldGVzLmRlZmF1bHQuc3Zjgh5rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNs
dXN0ZXKCJGt1YmVybmV0ZXMuZGVmYXVsdC5zdmMuY2x1c3Rlci5sb2NhbIcEfwAA
AYcEwKhqdYcEwKhq7YcEwKhqdocEwKhqd4cEwKhqgIcEwKhqgYcEwKhqZ4cEqakA
ATANBgkqhkiG9w0BAQsFAAOCAQEAUT/KK1VXLRBBy4tA6jIGb5T8m2Qmlhsh8LMI
0KjwplN0BR0t7lHvd7BHokR/dBgEZRTX3BowxWUgmZCH16q9BFwv1MSEs/wcCn6A
/TCPfA5skayhkL2FPZSQNoxHmGdjimYMGMcmdh6rrXBwzQZdfLHiIzmC1Q0XWIxz
Z5v6g2nV52NM9Ho3AMD43t1sl2BXya2CkrpjzKm3y7uAkMYzukxnXEDHuXcduROh
ngJX7dbWRZhd3OmdeuZRu+JbDMlJcY5kZHrErO5DaYRpdc2lcmfOJALTm40X5W/1
w3j63RnKtNsboBTSeqjyKYfbryBJQ5LYTu83LWoDckVWQa66YA==
-----END CERTIFICATE REQUEST-----
[root@managementa pki]# cat  kubernetes-key.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEA2Yhv0Je+Yxs9rqGkIBd3Jy1loq9bnHMgA8nVpu//QVR4n8Fq
q3Lvt3/XnJg36u/l0UufVQD/hd+MWKL/bqnJMYb5YGBZCd1P5VOE3YyKmIgiea79
y1XUjl5Bvn8i5ReQINT3RIrXqdw0Qm/bSSqvopTC6VPSN8Ax6eI0Lnr1sLdTH6zA
1UsUtKCHmM0Wh92PF158dihjy/YrtjMFe91P9dOJocbwwARzxJlRO21OSJC4P/zk
pDWBtd8Oxjs5UIkjXI6QDxx/cStGP2lJAm39Wgu2gg835gglhD8oeVnwfbRl52BB
dKmH2UL/Nb/oVkUVHK1W6bosFqE3SLDHmhPc9QIDAQABAoIBAQCpNvzj4mZzaald
wteNLzO9Ag9hsc8tsFBjIgpUxbRl+XOrsiVsIQhgUc5DPhWhZ+P6Hz1ePlyGoxLl
kEXqq6CaKkiqs8gPaFzSI1njjYPyi1NmHL3IAohKBwBVU0ittNqk74U5iFejBmyQ
kbqe+9mMOvQz1MReId+x9Ahrb7LXNxP9OtbuM5EUgMTFlsfrrgtNNJN0lg/BbmJn
mmPBe3MrEBr1EW5I/ZfWXdQcjqILxZEJZ9TfnB77Fj2gmNrIFxBKPNvn/Kl0ek0f
xFHfIU934aLCg/Ywmts+9BcbtKtBnHHOd6gnCB5dKgwl8/qKJfafCPhcraij8GKr
WmB3tXCxAoGBAP6G0BIJW91OY7tAgfRwuplYzFfV+Tx0LTCz5Gmh3DawWSoex6Hr
0gVo8Pm9wcEOSwXBUMuuMRxgFGWFZEazbDM4IuidVLcJIR2czmrU3gf0t+D1mkOf
N5+lMMOHE35cFcO6hHFnf/n7kkAN0RHxiO9+aVOlPe1diWdho0kkLzYjAoGBANrK
zW+9j2iHtks9BL4/wuRp6F7KMndZZr07z2fJ2+AiIThMHpsLVaNi1xH10AkRVpsZ
c/cEmEc/86I1dVTUvis/7tjaPxaYwA1jQ/yC1TP/tI57h8/MCrGR7ZplpuzymV+x
7E98TGq/qMky+tOXP9580vMwsK8JoktLRzPuCjYHAoGBALfY1O5SSELAXpVg8P2J
d59QXrmLWy3plMK7Dd+nBJOUKbOc7AHvfpJdzMH36L3z/wi3LA8TUXH3jIQQJ/BR
pXQRtlVjX0+ejob/PrI38/C3OSKLBNSXauwru99f8BqzlRz92rC3W99LccZGtJ9L
Yefr3VSH5QVRLPC5u+IW+usVAoGBAMWL2SOsDyD9cB3M0UyJu4mLCnETta9HPFld
+G2ot+tORZpUOEobWM51/uRLgvO9AOp3d9ov/uJOHsd15yOaFr5sMlb/73iSoM01
tHv5EVGq7ja72KtJetpLfTIr2CUXAl6CAnDeNQ0pUdegPRLw/I0BPWKwssbINw4u
wPJlWjjfAoGAVaDYwBeDHLbrzHYEJ8FejDT8h6UgeOL7sw3ua+OQjj5gM+/UlQZh
fpSIL7h93Zc3f7LBcDxo60THWDotm6TfLqu1RMco49FpxN6/3naqDjwL9t73b2uz
XdyCq5lI6PHQzy7PlVLP/TWzZW1YT4DpkRAc7S1zyGQxImW1UzZ4oO4=
-----END RSA PRIVATE KEY-----
[root@managementa pki]# cat  kubernetes.pem
-----BEGIN CERTIFICATE-----
MIIEnTCCA4WgAwIBAgIUXkAeoDoFReJbakmRKjvWJji/W+0wDQYJKoZIhvcNAQEL
BQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0Jl
aUppbmcxDDAKBgNVBAoTA2s4czEPMA0GA1UECxMGU3lzdGVtMRMwEQYDVQQDEwpr
dWJlcm5ldGVzMB4XDTE5MTExMzA3MjUwMFoXDTI5MTExMDA3MjUwMFowZTELMAkG
A1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0JlaUppbmcxDDAK
BgNVBAoTA2s4czEPMA0GA1UECxMGU3lzdGVtMRMwEQYDVQQDEwprdWJlcm5ldGVz
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA2Yhv0Je+Yxs9rqGkIBd3
Jy1loq9bnHMgA8nVpu//QVR4n8Fqq3Lvt3/XnJg36u/l0UufVQD/hd+MWKL/bqnJ
MYb5YGBZCd1P5VOE3YyKmIgiea79y1XUjl5Bvn8i5ReQINT3RIrXqdw0Qm/bSSqv
opTC6VPSN8Ax6eI0Lnr1sLdTH6zA1UsUtKCHmM0Wh92PF158dihjy/YrtjMFe91P
9dOJocbwwARzxJlRO21OSJC4P/zkpDWBtd8Oxjs5UIkjXI6QDxx/cStGP2lJAm39
Wgu2gg835gglhD8oeVnwfbRl52BBdKmH2UL/Nb/oVkUVHK1W6bosFqE3SLDHmhPc
9QIDAQABo4IBQzCCAT8wDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUF
BwMBBggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBTr2g9b5ihjJg1q
5D8XCjX2yE9I5jAfBgNVHSMEGDAWgBTANZLNOXsJnvLhswAZJ4sut9SbNjCBvwYD
VR0RBIG3MIG0ggprdWJlcm5ldGVzghJrdWJlcm5ldGVzLmRlZmF1bHSCFmt1YmVy
bmV0ZXMuZGVmYXVsdC5zdmOCHmt1YmVybmV0ZXMuZGVmYXVsdC5zdmMuY2x1c3Rl
coIka3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FshwR/AAABhwTA
qGp1hwTAqGrthwTAqGp2hwTAqGp3hwTAqGqAhwTAqGqBhwTAqGpnhwSpqQABMA0G
CSqGSIb3DQEBCwUAA4IBAQBibcKxXIvEu1RurTxjaaadTkO5lWU+6L2zar4KepdV
XQXiW+q1HvwVJl3ZM86I8xF8mp6nkH6SbUHx/tY8VT0WI7jC1wohUleli1AtnljR
BU2Ngq7v6kw/aH4CuVYryVU4YmLTGAKX8ZCt36/0uZKDq94q3E8EijUJXSi8Ek/O
CCldw/bJ5tCCo1lcfJiioRikbcFXo4zzt1hiILADxmqfjMv/75gTmyqygb5zkBfd
4YQG+XBkTZRvGKPEQjQzlLe8Pt0c22EjAoD4HamF95+eblEiTlmavCN4uHt3bSb9
ud4AtKd8+HhpC5m5fvvTrxgsCDDc671uDaXbiyb09X2w
-----END CERTIFICATE-----
[root@managementa pki]# 
```

### k8s DNS安装

```
k8s集群部署完后第一件事就是要配置DNS服务，目前可选的方案有skydns, kube-dns, coredns
[root@managementa fk]# pwd
/root/fk
[root@managementa fk]# mkdir kube-dns
[root@managementa fk]# cd kube-dns/
[root@managementa kube-dns]# wget -O kube-dns.yaml https://raw.githubusercontent.com/kubernetes/kubernetes/release-1.16/cluster/addons/dns/kube-dns/kube-dns.yaml.base
[root@managementa kube-dns]# docker pull netonline/k8s-dns-dnsmasq-nanny-amd64:1.14.8
[root@managementa kube-dns]# docker pull netonline/k8s-dns-kube-dns-amd64:1.14.8
[root@managementa kube-dns]# docker pull netonline/k8s-dns-sidecar-amd64:1.14.8
[root@managementa kube-dns]# docker tag netonline/k8s-dns-dnsmasq-nanny-amd64:1.14.8 192.168.106.117/library/k8s-dns-dnsmasq-nanny-amd64:1.14.8
[root@managementa kube-dns]# docker tag netonline/k8s-dns-kube-dns-amd64:1.14.8 192.168.106.117/library/k8s-dns-kube-dns-amd64:1.14.8
[root@managementa kube-dns]# docker tag netonline/k8s-dns-sidecar-amd64:1.14.8  192.168.106.117/library/k8s-dns-sidecar-amd64:1.14.8
[root@managementa kube-dns]# docker push 192.168.106.117/library/k8s-dns-dnsmasq-nanny-amd64:1.14.8
[root@managementa kube-dns]# docker push 192.168.106.117/library/k8s-dns-kube-dns-amd64:1.14.8
[root@managementa kube-dns]# docker push 192.168.106.117/library/k8s-dns-sidecar-amd64:1.14.8 

[root@managementa kube-dns]# docker rmi  netonline/k8s-dns-dnsmasq-nanny-amd64:1.14.8
[root@managementa kube-dns]# docker rmi netonline/k8s-dns-kube-dns-amd64:1.14.8
[root@managementa kube-dns]#  docker rmi netonline/k8s-dns-sidecar-amd64:1.14.8
[root@managementa kube-dns]# 
**修改yaml配置**
[root@managementa kube-dns]# cat kube-dns.yaml 
# Copyright 2016 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Should keep target in cluster/addons/dns-horizontal-autoscaler/dns-horizontal-autoscaler.yaml
# in sync with this file.

# __MACHINE_GENERATED_WARNING__

apiVersion: v1
kind: Service
metadata:
  name: kube-dns
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/name: "KubeDNS"
spec:
  selector:
    k8s-app: kube-dns
  ### 修改的
  clusterIP: 169.169.169.169
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
    protocol: TCP
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-dns
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-dns
  namespace: kube-system
  labels:
    addonmanager.kubernetes.io/mode: EnsureExists
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-dns
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  # replicas: not specified here:
  # 1. In order to make Addon Manager do not reconcile this replicas parameter.
  # 2. Default is 1.
  # 3. Will be tuned in real time if DNS horizontal auto-scaling is turned on.
  strategy:
    rollingUpdate:
      maxSurge: 10%
      maxUnavailable: 0
  selector:
    matchLabels:
      k8s-app: kube-dns
  template:
    metadata:
      labels:
        k8s-app: kube-dns
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: 'docker/default'
        prometheus.io/port: "10054"
        prometheus.io/scrape: "true"
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        supplementalGroups: [ 65534 ]
        fsGroup: 65534
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
      volumes:
      - name: kube-dns-config
        configMap:
          name: kube-dns
          optional: true
      containers:
      - name: kubedns
        image: 192.168.106.117/library/k8s-dns-kube-dns-amd64:1.14.8
        resources:
          # TODO: Set memory limits when we've profiled the container for large
          # clusters, then set request = limit to keep this container in
          # guaranteed class. Currently, this container falls into the
          # "burstable" category so the kubelet doesn't backoff from restarting it.
          limits:
            ###修改的
            memory: 170Mi
          requests:
            cpu: 100m
            memory: 70Mi
        livenessProbe:
          httpGet:
            path: /healthcheck/kubedns
            port: 10054
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8081
            scheme: HTTP
          # we poll on pod startup for the Kubernetes master service and
          # only setup the /readiness HTTP server once that's available.
          initialDelaySeconds: 3
          timeoutSeconds: 5
        args:
        - --domain=cluster.local.
        - --dns-port=10053
        - --config-dir=/kube-dns-config
        - --v=2
        env:
        - name: PROMETHEUS_PORT
          value: "10055"
        ports:
        - containerPort: 10053
          name: dns-local
          protocol: UDP
        - containerPort: 10053
          name: dns-tcp-local
          protocol: TCP
        - containerPort: 10055
          name: metrics
          protocol: TCP
        volumeMounts:
        - name: kube-dns-config
          mountPath: /kube-dns-config
      - name: dnsmasq
        image: 192.168.106.117/library/k8s-dns-dnsmasq-nanny-amd64:1.14.8
        livenessProbe:
          httpGet:
            path: /healthcheck/dnsmasq
            port: 10054
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        args:
        - -v=2
        - -logtostderr
        - -configDir=/etc/k8s/dns/dnsmasq-nanny
        - -restartDnsmasq=true
        - --
        - -k
        - --cache-size=1000
        - --no-negcache
        - --dns-loop-detect
        - --log-facility=-
        - --server=/cluster.local./127.0.0.1#10053
        - --server=/in-addr.arpa/127.0.0.1#10053
        - --server=/ip6.arpa/127.0.0.1#10053
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        # see: https://github.com/kubernetes/kubernetes/issues/29055 for details
        resources:
          requests:
            cpu: 150m
            memory: 20Mi
        volumeMounts:
        - name: kube-dns-config
          mountPath: /etc/k8s/dns/dnsmasq-nanny
      - name: sidecar
        image: 192.168.106.117/library/k8s-dns-sidecar-amd64:1.14.8
        livenessProbe:
          httpGet:
            path: /metrics
            port: 10054
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        args:
        - --v=2
        - --logtostderr
        - --probe=kubedns,127.0.0.1:10053,kubernetes.default.svc.cluster.local.,5,SRV
        - --probe=dnsmasq,127.0.0.1:53,kubernetes.default.svc.cluster.local.,5,SRV
        ports:
        - containerPort: 10054
          name: metrics
          protocol: TCP
        resources:
          requests:
            memory: 20Mi
            cpu: 10m
      dnsPolicy: Default  # Don't use cluster DNS.
      serviceAccountName: kube-dns
      
[root@managementa kube-dns]# kubectl create -f kube-dns.yaml
[root@managementa kube-dns]# kubectl get pod -n kube-system -o wide
[root@managementa kube-dns]# kubectl get service -n kube-system -o wide
[root@managementa kube-dns]# kubectl get deployment -n kube-system -o wide

验证
 如果我们此前已经有了部署了许多pod和服务，并且在Kubelet 启动配置文件加入了 --cluster-dns=169.169.169.169 --cluster-domain=cluster.local 
 
 dns验证
 root@managementa:~/fk/dns-test[root@managementa dns-test]# cat bsy-box.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: busybox
    role: master
  name: busybox
spec:
  containers:
  - name: busybox
    image: docker.io/busybox:latest
    command:
    - sleep
    - "3600"
```

### Dashboard 安装

```
安装v1.10.1有很多404错误，找不到heapster服务等错误，heapster已经被metrics server取代，可能是版本问题，安装最新版本。
GITHUB: https://github.com/kubernetes/dashboard
RELEASE: https://github.com/kubernetes/dashboard/releases

[root@managementa kube-dashboard]# wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta5/aio/deploy/recommended.yaml 

[root@managementa kube-dashboard]# cat recommended.yaml 
# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: v1
kind: Namespace
metadata:
  name: kubernetes-dashboard

---

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 31001
  selector:
    k8s-app: kubernetes-dashboard

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kubernetes-dashboard
type: Opaque

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-csrf
  namespace: kubernetes-dashboard
type: Opaque
data:
  csrf: ""

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-key-holder
  namespace: kubernetes-dashboard
type: Opaque

---

kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-settings
  namespace: kubernetes-dashboard

---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
rules:
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs", "kubernetes-dashboard-csrf"]
    verbs: ["get", "update", "delete"]
    # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["kubernetes-dashboard-settings"]
    verbs: ["get", "update"]
    # Allow Dashboard to get metrics.
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["heapster", "dashboard-metrics-scraper"]
    verbs: ["proxy"]
  - apiGroups: [""]
    resources: ["services/proxy"]
    resourceNames: ["heapster", "http:heapster:", "https:heapster:", "dashboard-metrics-scraper", "http:dashboard-metrics-scraper"]
    verbs: ["get"]

---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
rules:
  # Allow Metrics Scraper to get metrics from the Metrics server
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list", "watch"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
        - name: kubernetes-dashboard
          image: 192.168.106.117/library/kubernetesui/dashboard:v2.0.0-beta5
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            - --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            # Uncomment the following line to manually specify Kubernetes API server Host
            # If not specified, Dashboard will attempt to auto discover the API server and connect
            # to it. Uncomment only if the default does not work.
            # - --apiserver-host=http://my-address:port
          volumeMounts:
            - name: kubernetes-dashboard-certs
              mountPath: /certs
              # Create on-disk volume to store exec logs
            - mountPath: /tmp
              name: tmp-volume
          livenessProbe:
            httpGet:
              scheme: HTTPS
              path: /
              port: 8443
            initialDelaySeconds: 30
            timeoutSeconds: 30
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      volumes:
        - name: kubernetes-dashboard-certs
          secret:
            secretName: kubernetes-dashboard-certs
        - name: tmp-volume
          emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "beta.kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: dashboard-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 8000
      targetPort: 8000
  selector:
    k8s-app: dashboard-metrics-scraper

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: dashboard-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: dashboard-metrics-scraper
  template:
    metadata:
      labels:
        k8s-app: dashboard-metrics-scraper
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: 'runtime/default'
    spec:
      containers:
        - name: dashboard-metrics-scraper
          image: 192.168.106.117/library/metrics-scraper:v1.0.1
          ports:
            - containerPort: 8000
              protocol: TCP
          livenessProbe:
            httpGet:
              scheme: HTTP
              path: /
              port: 8000
            initialDelaySeconds: 30
            timeoutSeconds: 30
          volumeMounts:
          - mountPath: /tmp
            name: tmp-volume
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "beta.kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      volumes:
        - name: tmp-volume
          emptyDir: {}
          
          
          

本地下载image
[root@managementa kube-dashboard]# docker pull kubernetesui/dashboard:v2.0.0-beta5
[root@managementa kube-dashboard]# docker tag kubernetesui/dashboard:v2.0.0-beta5 192.168.106.117/library/kubernetesui/dashboard:v2.0.0-beta5
[root@managementa kube-dashboard]# docker push 192.168.106.117/library/kubernetesui/dashboard:v2.0.0-beta5
[root@managementa kube-dashboard]# docker rmi kubernetesui/dashboard:v2.0.0-beta5
修改image为:192.168.106.117/library/kubernetesui/dashboard:v2.0.0-beta5

[root@managementa kube-dashboard]# docker pull kubernetesui/metrics-scraper:v1.0.1
[root@managementa kube-dashboard]# docker tag kubernetesui/metrics-scraper:v1.0.1 192.168.106.117/library/metrics-scraper:v1.0.1
[root@managementa kube-dashboard]# docker push 192.168.106.117/library/metrics-scraper:v1.0.1
[root@managementa kube-dashboard]# docker rmi kubernetesui/metrics-scraper:v1.0.1
修改image为:192.168.106.117/library/metrics-scraper:v1.0.1

修改服务为NodePort，暴露服务

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 31001
  selector:
    k8s-app: kubernetes-dashboard

[root@managementa kube-dashboard]# kubectl create -f recommended.yaml 
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created




**创建dashboard管理员用户**

[root@managementa kube-dashboard]# cat dashboard-admin.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: dashboard-admin
  namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin-bind-cluster-role
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kubernetes-dashboard
[root@managementa kube-dashboard]# kubectl create -f dashboard-admin.yaml 
[root@managementa kube-dashboard]# kubectl get secrets --all-namespaces
NAMESPACE              NAME                                       TYPE                                  DATA   AGE
default                default-token-499vm                        kubernetes.io/service-account-token   3      18h
ingress-nginx          default-token-wqpw4                        kubernetes.io/service-account-token   3      11h
ingress-nginx          nginx-ingress-serviceaccount-token-xjvth   kubernetes.io/service-account-token   3      11h
kube-node-lease        default-token-brjgg                        kubernetes.io/service-account-token   3      18h
kube-public            default-token-944rr                        kubernetes.io/service-account-token   3      18h
kube-system            default-token-d6z59                        kubernetes.io/service-account-token   3      18h
kube-system            kube-dns-token-xh646                       kubernetes.io/service-account-token   3      18h
kube-system            kubernetes-dashboard-key-holder            Opaque                                2      18h
kube-system            metrics-server-token-sxsg6                 kubernetes.io/service-account-token   3      10h
kubernetes-dashboard   dashboard-admin-token-pk4vg                kubernetes.io/service-account-token   3      12s
kubernetes-dashboard   default-token-76s42                        kubernetes.io/service-account-token   3      29m
kubernetes-dashboard   kubernetes-dashboard-certs                 Opaque                                0      29m
kubernetes-dashboard   kubernetes-dashboard-csrf                  Opaque                                1      29m
kubernetes-dashboard   kubernetes-dashboard-key-holder            Opaque                                2      29m
kubernetes-dashboard   kubernetes-dashboard-token-b5r9b           kubernetes.io/service-account-token   3      29m

[root@managementa kube-dashboard]# kubectl describe secrets dashboard-admin-token-pk4vg -n kubernetes-dashboard 
Name:         dashboard-admin-token-pk4vg
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: adf25208-0634-4395-9e20-ad1dc5721cf3

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1359 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlhZLVNXNFhUM3ZWeW56M1hWYk5GRm9zZU9HZ1dBbk5ZVzM5VUhmbURaU1UifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tcGs0dmciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYWRmMjUyMDgtMDYzNC00Mzk1LTllMjAtYWQxZGM1NzIxY2YzIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmRhc2hib2FyZC1hZG1pbiJ9.QKKFKcD1sPxnoTv24bd7CQKXm1QNwfHiRGOhs-1ZluI4xDzsZsDg3ju-9wYFK5dQqtVnhQAqhovisUFt0tU6QHA0bcqJf4wcP5oyHUH2m4ZiKua5WjViFpaOTTaS8g_3q0yzoHBeTecc7LiBRu9pi38SHonZpz6pePKfBSqJPslj111OSwYcTKR5iLx5dsVkil80y04SNWlxP-mQvrBACIrAgxYAeGQMHvyoZ88ScQoMJR_jSUkXsg-SG9JJhY4JFyu4o_0oETRR0xP7FfytYAagUkL3ffXmaBzf1h0IkMrSYUbi7w_mcWMDjJhyavNI2kO1w9AQB6eXjL2BR2bAUw
```
### metrics server安装(K8S从1.7版本开始引入metrics server替代heapster)

```
apiserver开启如下参数
--requestheader-client-ca-file=
--proxy-client-cert-file=
--proxy-client-key-file=
--requestheader-allowed-names=
--requestheader-extra-headers-prefix=X-Remote-Extra-
--requestheader-group-headers=X-Remote-Group
--requestheader-username-headers=X-Remote-User
--enable-aggregator-routing=true

==============================================================================================
我的环境如下配置
--requestheader-client-ca-file=/root/fk/pki/ca.pem --proxy-client-cert-file=/root/fk/pki/kubernetes.pem --proxy-client-key-file=/root/fk/pki/kubernetes-key.pem
--requestheader-allowed-names=
--requestheader-extra-headers-prefix=X-Remote-Extra-
--requestheader-group-headers=X-Remote-Group
--requestheader-username-headers=X-Remote-User
--enable-aggregator-routing=true
===============================================================================================

下载地址https://github.com/kubernetes-incubator/metrics-server/
[root@managementa kube-metrics-server]# wget https://raw.githubusercontent.com/kubernetes-sigs/metrics-server/master/deploy/1.8%2B/aggregated-metrics-reader.yaml
[root@managementa kube-metrics-server]# wget https://raw.githubusercontent.com/kubernetes-sigs/metrics-server/master/deploy/1.8%2B/auth-delegator.yaml
[root@managementa kube-metrics-server]# wget https://raw.githubusercontent.com/kubernetes-sigs/metrics-server/master/deploy/1.8%2B/auth-reader.yaml
[root@managementa kube-metrics-server]# wget https://raw.githubusercontent.com/kubernetes-sigs/metrics-server/master/deploy/1.8%2B/metrics-apiservice.yaml
[root@managementa kube-metrics-server]# wget https://raw.githubusercontent.com/kubernetes-sigs/metrics-server/master/deploy/1.8%2B/metrics-server-deployment.yaml
[root@managementa kube-metrics-server]# wget https://raw.githubusercontent.com/kubernetes-sigs/metrics-server/master/deploy/1.8%2B/metrics-server-service.yaml
[root@managementa kube-metrics-server]# wget https://raw.githubusercontent.com/kubernetes-sigs/metrics-server/master/deploy/1.8%2B/resource-reader.yaml
[root@managementa kube-metrics-server]# docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
v0.3.6: Pulling from google_containers/metrics-server-amd64
e8d8785a314f: Pull complete 
b2f4b24bed0d: Pull complete 
Digest: sha256:c9c4e95068b51d6b33a9dccc61875df07dc650abbf4ac1a19d58b4628f89288b
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
[root@managementa kube-metrics-server]# docker tag   registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6 192.168.106.117/library/metrics-server-amd64:v0.3.6
[root@managementa kube-metrics-server]# docker push 192.168.106.117/library/metrics-server-amd64:v0.3.6
The push refers to repository [192.168.106.117/library/metrics-server-amd64]
7bf3709d22bb: Pushed 
932da5156413: Pushed 
v0.3.6: digest: sha256:c9c4e95068b51d6b33a9dccc61875df07dc650abbf4ac1a19d58b4628f89288b size: 738
[root@managementa kube-metrics-server]# docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64@sha256:c9c4e95068b51d6b33a9dccc61875df07dc650abbf4ac1a19d58b4628f89288b
[root@managementa kube-metrics-server]# 
修改metrics-server-deployment.yaml，添加command和修改imagePullPolicy
[root@managementa kube-metrics-server]# vi metrics-server-deployment.yaml 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      # mount in tmp so we can safely use from-scratch images and/or read-only containers
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        image: 192.168.106.117/library/metrics-server-amd64:v0.3.6
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
        ports:
        - name: main-port
          containerPort: 4443
          protocol: TCP
        securityContext:
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        imagePullPolicy: IfNotPresent
        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
          
下面的文件是不用修改的，只罗列内容
[root@managementa kube-metrics-server]# cat aggregated-metrics-reader.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:aggregated-metrics-reader
  labels:
    rbac.authorization.k8s.io/aggregate-to-view: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
rules:
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods", "nodes"]
  verbs: ["get", "list", "watch"]
  
[root@managementa kube-metrics-server]# cat auth-reader.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
[root@managementa kube-metrics-server]# cat resource-reader.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats
  - namespaces
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
[root@managementa kube-metrics-server]# cat auth-delegator.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
[root@managementa kube-metrics-server]# cat metrics-apiservice.yaml
---
apiVersion: apiregistration.k8s.io/v1beta1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
spec:
  service:
    name: metrics-server
    namespace: kube-system
  group: metrics.k8s.io
  version: v1beta1
  insecureSkipTLSVerify: true
  groupPriorityMinimum: 100
  versionPriority: 100
[root@managementa kube-metrics-server]# cat metrics-server-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    kubernetes.io/name: "Metrics-server"
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    k8s-app: metrics-server
  ports:
  - port: 443
    protocol: TCP
    targetPort: main-port
[root@managementa kube-metrics-server]# 
[root@managementa kube-metrics-server]# kubectl apply -f .



apiserver 加入参数 ： --enable-aggregator-routing ==》将聚合器请求路由到端点IP而不是集群IP。

root@managementa:~/fk/kube-metrics-server[root@managementa kube-metrics-server]# kubectl top nodes
NAME              CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
192.168.106.103   26m          0%     1397Mi          18%       
192.168.106.118   639m         15%    3068Mi          83%       
192.168.106.119   64m          1%     2551Mi          69%       
192.168.106.128   27m          0%     1403Mi          38%       
192.168.106.129   21m          0%     1442Mi          39%    
```

### Ingress安装

```
**apiserver配置**
--authorization-mode=RBAC

在https://github.com/kubernetes/ingress-nginx.git下的deploy/static目录下的mandatory.yaml文件，现在的网址是：https://github.com/kubernetes/ingress-nginx/blob/master/deploy/static/mandatory.yaml 。



在互联网上下载image
[root@managementa ~]# docker pull quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1
[root@yth-test ~]# docker save -o nginx-ingress-controller quay.io/kubernetes-ingress-controller/nginx-ingress-controller

内网导入

[root@managementa ~]# docker load -i nginx-ingress-controller
[root@managementa ~]# docker tag quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1 192.168.106.117/library/nginx-ingress-controller:0.26.1 
[root@managementa ~]# docker push  192.168.106.117/library/nginx-ingress-controller:0.26.1
The push refers to repository [192.168.106.117/library/nginx-ingress-controller]
30b6afc750c8: Pushed 
177e60f33aba: Pushed 
2cea43e70d3a: Pushed 
c51e4b7a59da: Pushed 
82c8c9d646ea: Pushed 
617b955bc962: Pushed 
4a812d06ca7b: Pushed 
3f8f5020e91a: Pushed 
fff9425704ad: Pushed 
23e1268cd811: Pushed 
e6f1810f12b9: Pushed 
69f5d91486f1: Pushed 
0.26.1: digest: sha256:5da1b2e84ecbdb27facbea84bc6ddc9d50145d824963230735b47828891cba7b size: 2839



修改image为192.168.106.117/library/nginx-ingress-controller:0.26.1
**和一些其他参数的修改**，经过修改后的文件内容是：

apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      # wait up to five minutes for the drain of connections
      #新加
      hostNetwork: true
      #如果已安装kube-dns那么优先使用k8s的dns，重点
      dnsPolicy: ClusterFirstWithHostNet
      terminationGracePeriodSeconds: 300
      serviceAccountName: nginx-ingress-serviceaccount
      nodeSelector:
        kubernetes.io/os: linux
      containers:
        - name: nginx-ingress-controller
          image:  192.168.106.117/library/nginx-ingress-controller:0.26.1
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
            #新加apiserver
            - --apiserver-host=http://192.168.106.117:9090
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            #新加变量
            - name: KUBERNETES_MASTER
              value: http://192.168.106.117:9090
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          lifecycle:
            preStop:
              exec:
                command:
                  - /wait-shutdown

---

[root@managementa workdir]# kubectl create -f mandaroty.yaml 
namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
deployment.apps/nginx-ingress-controller created

```
#### 错误解决

* Nov  8 12:49:33 managementa kube-controller-manager: E1108 12:49:33.842331   30295 replica_set.go:450] Sync "ingress-nginx/nginx-ingress-controller-579bdfc57f" failed with pods "nginx-ingress-controller-579bdfc57f-x9v54" is forbidden: SecurityContext.RunAsUser is forbidden
```
[root@managementa conf]# cat apiserver 
KUBE_API_ARGS="--service-account-key-file=/root/fk/conf/serviceaccount.key --etcd-servers=http://192.168.106.237:2379 --insecure-bind-address=0.0.0.0 --insecure-port=9090 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2"

去掉SecurityContextDeny

[root@managementa conf]# cat apiserver 
KUBE_API_ARGS="--service-account-key-file=/root/fk/conf/serviceaccount.key --etcd-servers=http://192.168.106.237:2379 --insecure-bind-address=0.0.0.0 --insecure-port=9090 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2"

重启apiserver
[root@managementa conf]# systemctl restart kube-apiserver

```
* E1108 05:13:57.600996       6 config.go:428] Expected to load root CA config from /var/run/secrets/kubernetes.io/serviceaccount/ca.crt, but got err: open /var/run/secrets/kubernetes.io/serviceaccount/ca.crt: no such file or directory
这个估计是没有指定api-server参数导致的，他默认使用https

```
[root@managementa ~]# kubectl get secrets --namespace=kube-system
NAME                  TYPE                                  DATA   AGE
default-token-w7j7z   kubernetes.io/service-account-token   2      3d3h
[root@managementa ~]# kubectl describe secret default-token-w7j7z --namespace=kube-system
Name:         default-token-w7j7z
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 1c90a9c9-cc82-48cd-b740-dce6f717743f

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImNUOUx4V2hVR0FtQ1J5V08wWnQxSU5mTEFsRzZrMUdDZ1drZTNYQllmVTAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLXc3ajd6Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIxYzkwYTljOS1jYzgyLTQ4Y2QtYjc0MC1kY2U2ZjcxNzc0M2YiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZGVmYXVsdCJ9.JrT-8Pj_ZhRndd8hb4wngBf7BEWWcLxjvGmNGotkKdhkPz2fKk-IUzoMqvG90rgOe70j1Ru5W0R5p6G0ryOBNzMZovBhZ3Na4Fhr8nJO8vAag995QkRxqek0LCyaAG1Vl7drxhnNgQUG-7W-TlZPs4XtfjDQDkQeNr87B5A-VOKd7vjWDKDdLY_XDFLvh4ykA-NO_ZJAegi21I3yvbbcCJNzWQwwLwoIFZiM8o5DMouVPOlsqbazs-svfp0y933YUOjUp9iR6IiNVQjAlqc1_RNpATVfgu9wzNlbwUOI6F8h4hXh-PH6Wg4GKcU1xGugskZhoQbJ5iBoUfa8CWxCBg
[root@managementa ~]# curl -L -O https://storage.googleapis.com/kubernetes-release/easy-rsa/easy-rsa.tar.gz
[root@managementa ~]# tar -xzvf easy-rsa.tar.gz 
[root@managementa ~]# cd easy-rsa-master/easyrsa3/
[root@managementa easyrsa3]# ./easyrsa init-pki
init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /root/easy-rsa-master/easyrsa3/pki

[root@managementa easyrsa3]# ./easyrsa --batch "--req-cn=192.168.106.117@`date +%s`" build-ca nopass
Generating a 2048 bit RSA private key
............................+++
...................................................+++
writing new private key to '/root/easy-rsa-master/easyrsa3/pki/private/ca.key'
-----

[root@managementa easyrsa3]# ./easyrsa --subject-alt-name="IP:192.168.106.117" build-server-full server nopass
Generating a 2048 bit RSA private key
..............................+++
................+++
writing new private key to '/root/easy-rsa-master/easyrsa3/pki/private/server.key'
-----
Using configuration from /root/easy-rsa-master/easyrsa3/openssl-1.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :PRINTABLE:'server'
Certificate is to be certified until Nov  5 06:21:34 2029 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated
[root@managementa easyrsa3]# 

[root@managementa easyrsa3]# mkdir /root/fk/pki
[root@managementa easyrsa3]# cp pki/ca.crt pki/issued/server.crt pki/private/server.key /root/fk/pki/
[root@managementa easyrsa3]# 
[root@managementa easyrsa3]# chmod 644 /root/fk/pki/*
[root@managementa easyrsa3]# 
分别修改apiserver和controller-manager配置文件，添加如下参数
KUBE_API_ARGS="--client-ca-file=/root/fk/pki/ca.crt --tls-cert-file=/root/fk/pki/server.crt --tls-private-key-file=/root/fk/pki/server.key"

KUBE_CONTROLLER_MANAGER_ARGS="--service_account_private_key_file=/root/fk/pki/server.key --root-ca-file=/root/fk/pki/ca.crt"


[root@managementa conf]# cat apiserver 
KUBE_API_ARGS="--service-account-key-file=/root/fk/conf/serviceaccount.key --etcd-servers=http://192.168.106.237:2379 --insecure-bind-address=0.0.0.0 --insecure-port=9090 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2 --client-ca-file=/root/fk/pki/ca.crt --tls-cert-file=/root/fk/pki/server.crt --tls-private-key-file=/root/fk/pki/server.key"

[root@managementa conf]# cat controller-manager 
KUBE_CONTROLLER_MANAGER_ARGS="--service-account-private-key-file=/root/fk/conf/serviceaccount.key --master=http://127.0.0.1:9090 --logtostderr=false --log-dir=/var/log/kubernetes --v=2 --service-account-private-key-file=/root/fk/pki/server.key --root-ca-file=/root/fk/pki/ca.crt" 

[root@managementa conf]# 
先不要重启apiserver和controller-manager
[root@managementa conf]# kubectl get secrets --all-namespaces
NAMESPACE         NAME                                       TYPE                                  DATA   AGE
default           default-token-9qp87                        kubernetes.io/service-account-token   2      3d5h
ingress-nginx     default-token-nl2c7                        kubernetes.io/service-account-token   2      4h15m
ingress-nginx     nginx-ingress-serviceaccount-token-kg58m   kubernetes.io/service-account-token   2      4h15m
kube-node-lease   default-token-6pnnc                        kubernetes.io/service-account-token   2      3d5h
kube-public       default-token-p8fnv                        kubernetes.io/service-account-token   2      3d5h
kube-system       default-token-w7j7z                        kubernetes.io/service-account-token   2      3d5h
[root@managementa conf]# systemctl stop kube-controller-manager
[root@managementa conf]# kubectl delete secret default-token-9qp87
secret "default-token-9qp87" deleted
[root@managementa conf]# kubectl delete secret default-token-nl2c7 --namespace=ingress-nginx
secret "default-token-nl2c7" deleted
[root@managementa conf]# kubectl delete secret default-token-6pnnc --namespace=kube-node-lease
secret "default-token-6pnnc" deleted
[root@managementa conf]# kubectl delete secret default-token-p8fnv --namespace=kube-public
secret "default-token-p8fnv" deleted
[root@managementa conf]# kubectl delete secret default-token-w7j7z --namespace=kube-system
secret "default-token-w7j7z" deleted
[root@managementa conf]# kubectl get secrets --all-namespaces
NAMESPACE       NAME                                       TYPE                                  DATA   AGE
ingress-nginx   nginx-ingress-serviceaccount-token-kg58m   kubernetes.io/service-account-token   2      4h18m
[root@managementa conf]# 
[root@managementa conf]# systemctl restart kube-apiserver
[root@managementa conf]# systemctl start  kube-controller-manager


[root@managementa conf]# kubectl get secrets --all-namespaces
NAMESPACE         NAME                                       TYPE                                  DATA   AGE
default           default-token-bb7jq                        kubernetes.io/service-account-token   3      22s
ingress-nginx     default-token-7k8jl                        kubernetes.io/service-account-token   3      22s
ingress-nginx     nginx-ingress-serviceaccount-token-kg58m   kubernetes.io/service-account-token   3      4h22m
kube-node-lease   default-token-hk4ck                        kubernetes.io/service-account-token   3      22s
kube-public       default-token-hkmpx                        kubernetes.io/service-account-token   3      22s
kube-system       default-token-z5mzv                        kubernetes.io/service-account-token   3      22s
[root@managementa conf]# kubectl describe secrets default-token-hkmpx -n kube-public
Name:         default-token-hkmpx
Namespace:    kube-public
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 082c7ee9-7f38-4f12-8ee0-cc1d23ebfd37

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImJpVTZrLVVFa3ViUTcwOF9Dam5CX1ZNdDE5OWNrRG1yek52VHpiN2w4cEUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXB1YmxpYyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLWhrbXB4Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwODJjN2VlOS03ZjM4LTRmMTItOGVlMC1jYzFkMjNlYmZkMzciLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1wdWJsaWM6ZGVmYXVsdCJ9.eefwvqPYb22ydbDWbCVElLXqHG_xU_qsJ7ucLs5Ths1v1PDksK-PR3RKHv9a88LWp8zgxB-VoyFP95koI3V7-NsZjhKww6I7Y0g4NqnWw7G1x2GMd7nW0KpElFW0kN5gI-0cXz-cFLbypOZiYEbkzQsSDmeMwnycKwfyjIa5Yqltyc3Dkh-3Hcy3adu0K0AhzDJR1Wo3lfsOTYF4FhKn4teAfL3BmhMZq_usgj5fPq9kLh41dEh5J0Eo4F7h3aE8sYVfJl5vq02T7ZC4N5kMHEb0ZJsqP8CdSvDxd3S2ttRDjXh8UG0qnNBLz6SJ1XHNPOIMFZMzaXp25WGhLAwFXQ
ca.crt:     1233 bytes


重建
[root@managementa workdir]# kubectl delete -f mandaroty.yaml 
namespace "ingress-nginx" deleted
configmap "nginx-configuration" deleted
configmap "tcp-services" deleted
configmap "udp-services" deleted
serviceaccount "nginx-ingress-serviceaccount" deleted
clusterrole.rbac.authorization.k8s.io "nginx-ingress-clusterrole" deleted
role.rbac.authorization.k8s.io "nginx-ingress-role" deleted
rolebinding.rbac.authorization.k8s.io "nginx-ingress-role-nisa-binding" deleted
clusterrolebinding.rbac.authorization.k8s.io "nginx-ingress-clusterrole-nisa-binding" deleted
deployment.apps "nginx-ingress-controller" deleted
[root@managementa workdir]# kubectl create -f mandaroty.yaml       
namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
deployment.apps/nginx-ingress-controller created
[root@managementa workdir]# 
```


#### ingress服务安装

```
[root@managementa workdir]# vi ingress-nginx.yaml
##vim ingress-nginx.yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      nodePort: 1080
      port: 80
      targetPort: 80
      protocol: TCP
    - name: https
      nodePort: 1443
      port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
[root@managementa workdir]# kubectl create -f ingress-nginx.yaml 
service/ingress-nginx created

安装完成

实例：
后端pod
[root@managementa ingress]# cat frontend-controller.yaml 
apiVersion: v1
kind: ReplicationController
metadata:
 name: frontend
 labels:
  name: frontend
spec:
 replicas: 3
 selector:
  name: frontend
 template:
  metadata:
   labels:
    name: frontend
  spec:
   containers:
   - name: frontend
     image: kubeguide/guestbook-php-frontend
     env:
     - name: GET_HOSTS_FROM
       value: env
     ports:
     - containerPort: 80
后端service
[root@managementa ingress]# cat frontend-service.yaml 
apiVersion: v1
kind: Service
metadata:
 name: frontend
 labels:
  name: frontend
spec:
 ports:
 - name: http
   port: 80
   targetPort: 80
   
 selector:
  name: frontend
创建ingress  
[root@managementa ingress]# cat ingress-frontend.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-frontend
  namespace: default
  annotations:
    kubernets.io/ingress.class: "nginx"
spec:
  rules:
  - http:
    #host: frontend.default.svc.cluster.local 访问这个服务时使用的域名，请组合nginx+server_name详细了解，nginx 根据 server_name 匹配 HTTP 请求头的 host，去决定使用那个 server。host的值浏览器默认填写域名。如果所有server_name都匹配不到就使用默认的server，所以写个这个是最靠谱的。
      paths:
      - path:
        backend:
          serviceName: frontend
          servicePort: 80

[root@managementa ingress]# 

```

### k8s测试环境配置

```
[root@managementa fk]# ls
bin  conf  dns-test  ingress  kube-dashboard  kube-dns  kube-metrics-server  pki
[root@managementa fk]# cd conf
[root@managementa conf]# ls
apiserver  bootstrap.kubeconfig  controller-manager  kubelet  proxy  scheduler
[root@managementa conf]# cat kubelet 
KUBELET_ARGS="--kubeconfig=/root/fk/conf/bootstrap.kubeconfig --cluster-dns=169.169.169.169 --cluster-domain=cluster.local --hostname-override=192.168.106.118 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"

[root@managementa conf]# cat proxy 
KUBE_PROXY_ARGS="--master=http://192.168.106.117:9090 --hostname-override=192.168.106.118 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
[root@managementa conf]# cat scheduler
KUBE_SCHEDULER_ARGS="--master=http://127.0.0.1:9090 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
[root@managementa conf]# cat bootstrap.kubeconfig
apiVersion: v1
clusters:
- cluster:
    server: http://192.168.106.117:9090
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubelet-bootstrap
  name: default
current-context: default
kind: Config
preferences: {}
users: []
[root@managementa conf]# cat controller-manager 
KUBE_CONTROLLER_MANAGER_ARGS="--master=http://127.0.0.1:9090 --logtostderr=false --log-dir=/var/log/kubernetes --v=2 --service-account-private-key-file=/root/fk/pki/kubernetes-key.pem --root-ca-file=/root/fk/pki/ca.pem" 
[root@managementa conf]# cat apiserver 
KUBE_API_ARGS="--allow-privileged=true --enable-aggregator-routing=true --authorization-mode=RBAC --etcd-servers=http://192.168.106.237:2379 --insecure-bind-address=0.0.0.0 --insecure-port=9090 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2 --client-ca-file=/root/fk/pki/ca.pem --tls-cert-file=/root/fk/pki/kubernetes.pem --tls-private-key-file=/root/fk/pki/kubernetes-key.pem --requestheader-client-ca-file=/root/fk/pki/ca.pem --proxy-client-cert-file=/root/fk/pki/kubernetes.pem --proxy-client-key-file=/root/fk/pki/kubernetes-key.pem --requestheader-allowed-names= --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User"

[root@managementa fk]# pwd
/root/fk
[root@managementa fk]# cat PodSecurityPolicy.yaml 
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: privileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: '*'
spec:
  privileged: true
  allowPrivilegeEscalation: true
  allowedCapabilities:
  - '*'
  volumes:
  - '*'
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  hostIPC: true
  hostPID: true
  runAsUser:
    rule: 'RunAsAny'
  seLinux:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'

```


