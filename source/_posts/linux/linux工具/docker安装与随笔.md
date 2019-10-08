---
title:  docker安装与随笔
tags:
- docker
categories: 
- linux 
date: 2019-09-17 02:00:00
---
> docker安装与随笔
<!-- more -->

## docker 安装
  [参照连接](https://docs.docker.com/install/linux/docker-ce/binaries/#install-static-binaries)
  
```
[root@iz2zegdp0r569zzor0c3quz ~]# tar -xzvf docker-19.03.0.tgz
[root@iz2zegdp0r569zzor0c3quz ~]# cd docker
[root@iz2zegdp0r569zzor0c3quz ~]# mv * /usr/bin/
如果不是默认path目录，那么请用systemctl show-environment/set-environment
[root@iz2zegdp0r569zzor0c3quz ~]# vi .bash_profile
[root@iz2zegdp0r569zzor0c3quz ~]# source .bash_profile
PATH=$PATH:$HOME/bin
[root@iz2zegdp0r569zzor0c3quz ~]# vi /etc/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target

[root@iz2zegdp0r569zzor0c3quz ~]# systemctl daemon-reload
[root@iz2zegdp0r569zzor0c3quz ~]# systemctl start docker
```

## docker-compose 安装

```
curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## dockerfile 制作镜像

```
[root@iz2zegdp0r569zzor0c3quz mk]# pwd
/root/mk
[root@iz2zegdp0r569zzor0c3quz mk]# cat Dockerfile 
FROM centos

MAINTAINER fengk

ENV DATA_PATH /data
ENV MYSQL_PASSWORD 123456
ENV PATH /usr/local/mysql/bin:$PATH
VOLUME [$DATA_PATH]

COPY my.sh /usr/local/
RUN cd /usr/local && chmod +x my.sh
COPY mysql-5.7.27-linux-glibc2.12-x86_64.tar.gz /usr/local/
RUN yum -y install numactl && yum -y install libaio
RUN groupadd mysql;useradd -r -g mysql -s /bin/false mysql
RUN cd /usr/local && tar -xzvf mysql-5.7.27-linux-glibc2.12-x86_64.tar.gz && ln -s mysql-5.7.27-linux-glibc2.12-x86_64 mysql && rm -fr mysql-5.7.27-linux-glibc2.12-x86_64.tar.gz
EXPOSE 3306
ENTRYPOINT ["/bin/sh","/usr/local/my.sh"]
[root@iz2zegdp0r569zzor0c3quz mk]# cat my.sh 
#!/usr/bin/bash
if [ ! -d $DATA_PATH ] ; then mkdir -p $DATA_PATH/data ; fi
if [ ! -d $DATA_PATH/data ] ; then mkdir $DATA_PATH/data ; fi
if [ ! -f $DATA_PATH/mariadb.log ] ; then touch $DATA_PATH/mariadb.log && chown mysql:mysql $DATA_PATH/mariadb.log ; fi
if [ ! -f $DATA_PATH/my.cnf ] ; then cd $DATA_PATH && echo -e "[mysqld]\nuser=mysql\ndatadir=${DATA_PATH}/data\nsocket=${DATA_PATH}/mysql.sock\n[mysqld_safe]\nlog-error=${DATA_PATH}/mariadb.log\npid-file=${DATA_PATH}/mariadb.pid" > my.cnf && chmod 664 my.cnf && chown -R mysql:mysql $DATA_PATH && cd /usr/local/mysql && ./bin/mysqld  --defaults-file=$DATA_PATH/my.cnf --initialize --user=mysql --basedir=/usr/local/mysql  --datadir=$DATA_PATH/data && cd /usr/local/mysql && ./bin/mysqld_safe --defaults-file=$DATA_PATH/my.cnf --skip-grant-tables &

var=1
until [ $var -eq 0 ]
do
sleep 1
/usr/local/mysql/bin/mysql -uroot -p123456 --socket=/data/mysql.sock -e "exit"
var=$?
done

cd /usr/local/mysql && ./bin/mysql -uroot --socket=/data/mysql.sock mysql -e "update user set authentication_string=password('$MYSQL_PASSWORD'),password_expired='N'"

cd /usr/local/mysql && ./bin/mysqladmin -u root --socket=$DATA_PATH/mysql.sock shutdown;  fi
cd /usr/local/mysql && ./bin/mysqld_safe --defaults-file=$DATA_PATH/my.cnf & 
/usr/bin/bash
[root@iz2zegdp0r569zzor0c3quz mk]# ls
Dockerfile  my.sh  mysql-5.7.27-linux-glibc2.12-x86_64.tar.gz
[root@iz2zegdp0r569zzor0c3quz mk]# docker build -t mysql .
[root@iz2zegdp0r569zzor0c3quz mk]# docker run -t -i -v /data:/data -p 3307:3306 --name db mysql
[root@iz2zegdp0r569zzor0c3quz mk]# docker run --link db:db -t -i centos (使用mysql -u*** -p*** -hdb访问上一个容器的mysql)
```

## etcd安装
  从github上下载etcd发布的二进制文件。
```
[root@iz2zegdp0r569zzor0c3quz ~]# vi /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target

[Service]
Type=simple
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd
[Install]
WantedBy=multi-user.target

[root@iz2zegdp0r569zzor0c3quz ~]#mkdir /var/lib/etcd/
[root@iz2zegdp0r569zzor0c3quz ~]#mkdir /etc/etcd/
[root@iz2zegdp0r569zzor0c3quz ~]#touch /etc/etcd/etcd.conf
[root@iz2zegdp0r569zzor0c3quz ~]#tar -xzvf etcd-v3.4.0-linux-amd64.tar.gz 
[root@iz2zegdp0r569zzor0c3quz ~]#cd etcd-v3.4.0-linux-amd64
[root@iz2zegdp0r569zzor0c3quz ~]#mv etcd etcdctl /usr/bin
[root@iz2zegdp0r569zzor0c3quz ~]# systemctl daemon-reload
[root@iz2zegdp0r569zzor0c3quz ~]# systemctl enable etcd
[root@iz2zegdp0r569zzor0c3quz ~]# systemctl start etcd
[root@iz2zegdp0r569zzor0c3quz ~]# systemctl status etcd
```




