---
title:  rancher安装
tags:
- 虚拟化
categories: 
- linux 
date: 2019-11-14 02:00:00
---
> rancher安装
<!-- more -->

### rancher安装

```
直接通过docker镜像来运行我们的rancher，首先，先从镜像中心下载rancher镜像，如果是1.x系列的，镜像名为rancher/server，而2.x是rancher/rancher，我们使用2.x版本的，所以，执行如下命令即可：
[root@managementa ~]# docker pull rancher/rancher
[root@managementa ~]# docker inspect rancher/rancher
[
    {
        "Id": "sha256:c583c217e1217ed75b465851999d44ad1f66b0c7c3cd596cd17b67f0728ff28b",
        "RepoTags": [
            "rancher/rancher:latest"
        ],
        "RepoDigests": [
            "rancher/rancher@sha256:f8751258c145cfa8cfb5e67d9784863c67937be3587c133288234a077ea386f4"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2019-10-28T23:32:55.872163521Z",
        "Container": "fcf6c7430250a086c3525b21156f5a69a8920a9dd1ed1d457b57e6aa2ff6ad66",
        "ContainerConfig": {
            "Hostname": "fcf6c7430250",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "CATTLE_SYSTEM_CHART_DEFAULT_BRANCH=release-v2.3",
                "CATTLE_HELM_VERSION=v2.14.3-rancher1",
                "CATTLE_K3S_VERSION=v0.8.0",
                "CATTLE_MACHINE_VERSION=v0.15.0-rancher12-1",
                "CATTLE_ETCD_VERSION=v3.3.14",
                "LOGLEVEL_VERSION=v0.1.2",
                "TINI_VERSION=v0.18.0",
                "TELEMETRY_VERSION=v0.5.10",
                "KUBECTL_VERSION=v1.16.1",
                "DOCKER_MACHINE_LINODE_VERSION=v0.1.8",
                "LINODE_UI_DRIVER_VERSION=v0.3.0",
                "TINI_URL_amd64=https://github.com/krallin/tini/releases/download/v0.18.0/tini",
                "TINI_URL_arm64=https://github.com/krallin/tini/releases/download/v0.18.0/tini-arm64",
                "TINI_URL=TINI_URL_amd64",
                "HELM_URL_amd64=https://github.com/rancher/helm/releases/download/v2.14.3-rancher1/rancher-helm",
                "HELM_URL_arm64=https://github.com/rancher/helm/releases/download/v2.14.3-rancher1/rancher-helm-arm64",
                "HELM_URL=HELM_URL_amd64",
                "TILLER_URL_amd64=https://github.com/rancher/helm/releases/download/v2.14.3-rancher1/rancher-tiller",
                "TILLER_URL_arm64=https://github.com/rancher/helm/releases/download/v2.14.3-rancher1/rancher-tiller-arm64",
                "TILLER_URL=TILLER_URL_amd64",
                "K3S_URL_amd64=https://github.com/rancher/k3s/releases/download/v0.8.0/k3s",
                "K3S_URL_arm64=https://github.com/rancher/k3s/releases/download/v0.8.0/k3s-arm64",
                "K3S_URL=K3S_URL_amd64",
                "ETCD_URL_amd64=https://github.com/etcd-io/etcd/releases/download/v3.3.14/etcd-v3.3.14-linux-amd64.tar.gz",
                "ETCD_URL_arm64=https://github.com/etcd-io/etcd/releases/download/v3.3.14/etcd-v3.3.14-linux-arm64.tar.gz",
                "ETCD_URL=ETCD_URL_amd64",
                "CATTLE_UI_PATH=/usr/share/rancher/ui",
                "CATTLE_UI_VERSION=2.3.22",
                "CATTLE_CLI_VERSION=v2.3.1",
                "CATTLE_API_UI_VERSION=1.1.6",
                "AUDIT_LOG_PATH=/var/log/auditlog/rancher-api-audit.log",
                "AUDIT_LOG_MAXAGE=10",
                "AUDIT_LOG_MAXBACKUP=10",
                "AUDIT_LOG_MAXSIZE=100",
                "AUDIT_LEVEL=0",
                "CATTLE_CLI_URL_DARWIN=https://releases.rancher.com/cli2/v2.3.1/rancher-darwin-amd64-v2.3.1.tar.gz",
                "CATTLE_CLI_URL_LINUX=https://releases.rancher.com/cli2/v2.3.1/rancher-linux-amd64-v2.3.1.tar.gz",
                "CATTLE_CLI_URL_WINDOWS=https://releases.rancher.com/cli2/v2.3.1/rancher-windows-386-v2.3.1.zip",
                "CATTLE_SERVER_VERSION=v2.3.2",
                "CATTLE_AGENT_IMAGE=rancher/rancher-agent:v2.3.2",
                "CATTLE_SERVER_IMAGE=rancher/rancher",
                "ETCD_UNSUPPORTED_ARCH=amd64",
                "ETCDCTL_API=3",
                "SSL_CERT_DIR=/etc/rancher/ssl"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "LABEL org.label-schema.vcs-url=https://github.com/rancher/rancher.git"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:e79cd9013af0d9fefe848328ecf7229fd868b14ac2e2ba6e8e06ffa0e6bfbffe",
            "Volumes": {
                "/var/lib/rancher": {},
                "/var/log/auditlog": {}
            },
            "WorkingDir": "/var/lib/rancher",
            "Entrypoint": [
                "entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "2019-10-28T23:31:40Z",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vcs-ref": "3324ee953540559d896391ba36c58b32c5e740e6",
                "org.label-schema.vcs-url": "https://github.com/rancher/rancher.git"
            }
        },
        "DockerVersion": "18.09.0",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "CATTLE_SYSTEM_CHART_DEFAULT_BRANCH=release-v2.3",
                "CATTLE_HELM_VERSION=v2.14.3-rancher1",
                "CATTLE_K3S_VERSION=v0.8.0",
                "CATTLE_MACHINE_VERSION=v0.15.0-rancher12-1",
                "CATTLE_ETCD_VERSION=v3.3.14",
                "LOGLEVEL_VERSION=v0.1.2",
                "TINI_VERSION=v0.18.0",
                "TELEMETRY_VERSION=v0.5.10",
                "KUBECTL_VERSION=v1.16.1",
                "DOCKER_MACHINE_LINODE_VERSION=v0.1.8",
                "LINODE_UI_DRIVER_VERSION=v0.3.0",
                "TINI_URL_amd64=https://github.com/krallin/tini/releases/download/v0.18.0/tini",
                "TINI_URL_arm64=https://github.com/krallin/tini/releases/download/v0.18.0/tini-arm64",
                "TINI_URL=TINI_URL_amd64",
                "HELM_URL_amd64=https://github.com/rancher/helm/releases/download/v2.14.3-rancher1/rancher-helm",
                "HELM_URL_arm64=https://github.com/rancher/helm/releases/download/v2.14.3-rancher1/rancher-helm-arm64",
                "HELM_URL=HELM_URL_amd64",
                "TILLER_URL_amd64=https://github.com/rancher/helm/releases/download/v2.14.3-rancher1/rancher-tiller",
                "TILLER_URL_arm64=https://github.com/rancher/helm/releases/download/v2.14.3-rancher1/rancher-tiller-arm64",
                "TILLER_URL=TILLER_URL_amd64",
                "K3S_URL_amd64=https://github.com/rancher/k3s/releases/download/v0.8.0/k3s",
                "K3S_URL_arm64=https://github.com/rancher/k3s/releases/download/v0.8.0/k3s-arm64",
                "K3S_URL=K3S_URL_amd64",
                "ETCD_URL_amd64=https://github.com/etcd-io/etcd/releases/download/v3.3.14/etcd-v3.3.14-linux-amd64.tar.gz",
                "ETCD_URL_arm64=https://github.com/etcd-io/etcd/releases/download/v3.3.14/etcd-v3.3.14-linux-arm64.tar.gz",
                "ETCD_URL=ETCD_URL_amd64",
                "CATTLE_UI_PATH=/usr/share/rancher/ui",
                "CATTLE_UI_VERSION=2.3.22",
                "CATTLE_CLI_VERSION=v2.3.1",
                "CATTLE_API_UI_VERSION=1.1.6",
                "AUDIT_LOG_PATH=/var/log/auditlog/rancher-api-audit.log",
                "AUDIT_LOG_MAXAGE=10",
                "AUDIT_LOG_MAXBACKUP=10",
                "AUDIT_LOG_MAXSIZE=100",
                "AUDIT_LEVEL=0",
                "CATTLE_CLI_URL_DARWIN=https://releases.rancher.com/cli2/v2.3.1/rancher-darwin-amd64-v2.3.1.tar.gz",
                "CATTLE_CLI_URL_LINUX=https://releases.rancher.com/cli2/v2.3.1/rancher-linux-amd64-v2.3.1.tar.gz",
                "CATTLE_CLI_URL_WINDOWS=https://releases.rancher.com/cli2/v2.3.1/rancher-windows-386-v2.3.1.zip",
                "CATTLE_SERVER_VERSION=v2.3.2",
                "CATTLE_AGENT_IMAGE=rancher/rancher-agent:v2.3.2",
                "CATTLE_SERVER_IMAGE=rancher/rancher",
                "ETCD_UNSUPPORTED_ARCH=amd64",
                "ETCDCTL_API=3",
                "SSL_CERT_DIR=/etc/rancher/ssl"
            ],
            "Cmd": null,
            "ArgsEscaped": true,
            "Image": "sha256:e79cd9013af0d9fefe848328ecf7229fd868b14ac2e2ba6e8e06ffa0e6bfbffe",
            "Volumes": {
                "/var/lib/rancher": {},
                "/var/log/auditlog": {}
            },
            "WorkingDir": "/var/lib/rancher",
            "Entrypoint": [
                "entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "2019-10-28T23:31:40Z",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vcs-ref": "3324ee953540559d896391ba36c58b32c5e740e6",
                "org.label-schema.vcs-url": "https://github.com/rancher/rancher.git"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 678399780,
        "VirtualSize": 678399780,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/14cb1b82c85d80403138698482f8f139631ef21bd4a2bcee021be6971425a544/diff:/var/lib/docker/overlay2/e25087faa5c72785842bd3e17fe1e139f310d4a98987150dc641dfdb6412ed94/diff:/var/lib/docker/overlay2/b737c202c76dd08f0f010b4db51032e0b480e4a7e87ccf31663ab160da9a7e24/diff:/var/lib/docker/overlay2/2bce0e22ee09bc8af4b2fa37aba818b4360fe086f02f28c32aee86b6e8d74642/diff:/var/lib/docker/overlay2/29fb96076e3449a233322a74025e3946817835ac5d9697545ac1e283f5382704/diff:/var/lib/docker/overlay2/3a71c30625cdf9cf0f6acc3adb5c3a5524cb10ae8aca4725577a34ddfae1b663/diff:/var/lib/docker/overlay2/7ed692f9b7b72a5815b8e7528a0fcea517c16d5fd1e507d0dc9bdd98eca568e4/diff:/var/lib/docker/overlay2/3f2f9950842e4ce6c5ebe737c709a25fc6e916211da80a5af15c71f606581b69/diff:/var/lib/docker/overlay2/aca0e6e5d49ed370178864c093497fb39425e0d1dd33dff9c6df8fd3173b4e4c/diff:/var/lib/docker/overlay2/2b7031e0aba2a918935f660b091380212774af8ca369f0926b1a34ef977c4056/diff:/var/lib/docker/overlay2/cc90fbdfd74ed3296cf1b53c8f203e6c2b0f93f2c284ef0f5cf4da420ad1f8d2/diff:/var/lib/docker/overlay2/70741c354abb79e9883de4e8bedabb4a1185fb36651b5f35bd3e88ac4dafeec6/diff:/var/lib/docker/overlay2/d57cc4ba6b11915263a51fcc74d4cc664b81a507f2eaed6d4c74fc76d41b8ab3/diff",
                "MergedDir": "/var/lib/docker/overlay2/b8e4c2447efe94c0783bc0945cec745e8f001dda4e5239d0f93a36d679ba88eb/merged",
                "UpperDir": "/var/lib/docker/overlay2/b8e4c2447efe94c0783bc0945cec745e8f001dda4e5239d0f93a36d679ba88eb/diff",
                "WorkDir": "/var/lib/docker/overlay2/b8e4c2447efe94c0783bc0945cec745e8f001dda4e5239d0f93a36d679ba88eb/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:a090697502b8d19fbc83afb24d8fb59b01e48bf87763a00ca55cfff42423ad36",
                "sha256:97e6b67a30f1efeb050ada13c2afa1afd748e175ae744027dd0cce1f2931a594",
                "sha256:100ef12ce3a46c3242d186dbbadedff1638dc1f69cab4e1fbf73489049c01c25",
                "sha256:19331eff40f01dd084a3f966cc6939e828d617d777163706b8a13d0f972704d1",
                "sha256:a666e243602f66e7ca489c227e4498601dae254e70aed790887fa3bd30881441",
                "sha256:336ad0c34c2add7870b8ed45f81c4a6edaf525f65ebee550079c97ab29a38844",
                "sha256:dd5139ba3fc6a921ee4cf2118cba2398acd66faea41205f3624464caf2a648ce",
                "sha256:bf848a426199e73d9fd9b9eb2d9ad507413034d378205cf7f3fa704916541912",
                "sha256:745847d679461acfb87a5ec54619c7a6b679de7e6deebf51ab580e2bf86c29b4",
                "sha256:e3f8d500bfb58c09617bee8464399ed91072396917f7a8c1f433c6aa167ccc7a",
                "sha256:e32a5a776af55d5e17c1ccc391c937566b8b55457b3ded813571a4a254d9b85e",
                "sha256:fb4b8e3e5f8e4ccd3277c7aa0f568dbc8e29858cf11b47855fbfc04e56768806",
                "sha256:98fdafb47746f9c3849fcc025d264aa3bcab69cb56ac45b2c73f440cb714c6fc",
                "sha256:2baec139ab419709e8e34786a1ecc41aa2fdcc3128570e10f2c62cd44b48095a"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
从中可以看出一些对我们有用的环境变量Env，以及数据卷Volumes等一般在Dockerfile构建中的参数。显而易见，rancher镜像主要有两个volume目录，默认方式是采用匿名卷的方式。接下来我们使用挂载到指定的主机目录方式来进行数据卷持久化同时启动rancher。
[root@managementa ~]# mkdir -p fk/rancher/rancher
[root@managementa ~]# mkdir -p fk/rancher/auditlog
[root@managementa ~]# docker tag rancher/rancher 192.168.106.117/library/rancher
[root@managementa ~]# docker push 192.168.106.117/library/rancher
[root@managementa ~]# docker rmi rancher/rancher
docker run -d --restart=unless-stopped -p 10080:80 -p 10443:443 -v /root/fk/rancher/rancher:/var/lib/rancher -v /root/fk/rancher/auditlog:/var/log/auditlog --name rancher 192.168.106.117/library/rancher

浏览器
https://192.168.106.117:10080
输入密码admin@123

curl --insecure -sfL https://192.168.106.117:10443/v3/import/8n4pm6t4bt7l4nj7sl4ttc9ldh28dcstkbhv2pcllp5bw25dgqm9vn.yaml | kubectl apply -f -
```
#### 错误

```
The DaemonSet "cattle-node-agent" is invalid: spec.template.spec.containers[0].securityContext.privileged: Forbidden: disallowed by cluster policy
解决:
1. 
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
    

[root@managementa fk]# kubectl create -f PodSecurityPolicy.yaml 
podsecuritypolicy.policy/privileged created
[root@managementa fk]# kubectl get PodSecurityPolicy
NAME         PRIV   CAPS   SELINUX    RUNASUSER   FSGROUP    SUPGROUP   READONLYROOTFS   VOLUMES
privileged   true   *      RunAsAny   RunAsAny    RunAsAny   RunAsAny   false  

2.apiserver指定--allow-privileged=true参数，仅设置apiserver即可
root@managementa:~/fk/conf[root@managementa conf]# pwd
/root/fk/conf
root@managementa:~/fk/conf[root@managementa conf]# cat apiserver 
KUBE_API_ARGS="--allow-privileged=true --enable-aggregator-routing=true --authorization-mode=RBAC --etcd-servers=http://192.168.106.237:2379 --insecure-bind-address=0.0.0.0 --insecure-port=9090 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2 --client-ca-file=/root/fk/pki/ca.pem --tls-cert-file=/root/fk/pki/kubernetes.pem --tls-private-key-file=/root/fk/pki/kubernetes-key.pem --requestheader-client-ca-file=/root/fk/pki/ca.pem --proxy-client-cert-file=/root/fk/pki/kubernetes.pem --proxy-client-key-file=/root/fk/pki/kubernetes-key.pem --requestheader-allowed-names= --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User"
```


#### 删除时存在一直 Terminating解决

问题在于metadata中存在 Finalizers
    k8s 资源的 metadata 里如果存在 finalizers，那么该资源一般是由某程序创建的，并且在其创建的资源的 metadata 里的 finalizers 加了一个它的标识，这意味着这个资源被删除时需要由创建资源的程序来做删除前的清理，清理完了它需要将标识从该资源的 finalizers 中移除，然后才会最终彻底删除资源。比如 Rancher 创建的一些资源就会写入 finalizers 标识。
处理建议：kubectl edit 手动编辑资源定义，删掉 finalizers，这时再看下资源，就会发现已经删掉了。