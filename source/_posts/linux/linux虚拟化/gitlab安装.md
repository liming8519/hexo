---
title:  gitlab安装
tags:
- 虚拟化
categories: 
- linux 
date: 2019-11-19 02:00:00
---
> gitlab安装
<!-- more -->

### gitlab安装

```
例子中安装的是11.11.3版本，此版本和下面的gitlab-runner结合出现job一直running的情况，换成12.4.1版本则没有此问题。可能是gitlab-runner的版本是12.4.1与11.11.3的gitlab不匹配导致的
root@yth-test:~[root@yth-test ~]# yum install -y curl policycoreutils-python openssh-server
root@yth-test:~[root@yth-test ~]# getenforce 
root@yth-test:~[root@yth-test ~]# wget -O gitlab.11.11.3.rpm https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-11.11.3-ce.0.el7.x86_64.rpm/download.rpm
root@yth-test:~[root@yth-test ~]# rpm -ivh gitlab.11.11.3.rpm
external_url 'http://39.106.96.125:65535'
nginx['listen_port'] = 65535
unicorn['port'] = 65534
root@yth-test:~[root@yth-test ~]# gitlab-ctl reconfigure
root@yth-test:~[root@yth-test ~]# gitlab-ctl start
ok: run: alertmanager: (pid 24224) 32s
ok: run: gitaly: (pid 24027) 35s
ok: run: gitlab-monitor: (pid 24095) 34s
ok: run: gitlab-workhorse: (pid 24061) 35s
ok: run: logrotate: (pid 23592) 86s
ok: run: nginx: (pid 23543) 92s
ok: run: node-exporter: (pid 24081) 34s
ok: run: postgres-exporter: (pid 24240) 31s
ok: run: postgresql: (pid 23185) 141s
ok: run: prometheus: (pid 24115) 33s
ok: run: redis: (pid 22967) 148s
ok: run: redis-exporter: (pid 24103) 33s
ok: run: sidekiq: (pid 23477) 104s
ok: run: unicorn: (pid 23420) 110s
root@yth-test:~[root@yth-test ~]# gitlab-ctl status
run: alertmanager: (pid 24224) 39s; run: log: (pid 23949) 60s
run: gitaly: (pid 24027) 42s; run: log: (pid 23074) 152s
run: gitlab-monitor: (pid 24095) 41s; run: log: (pid 23770) 78s
run: gitlab-workhorse: (pid 24061) 42s; run: log: (pid 23524) 104s
run: logrotate: (pid 23592) 93s; run: log: (pid 23608) 92s
run: nginx: (pid 23543) 99s; run: log: (pid 23566) 96s
run: node-exporter: (pid 24081) 41s; run: log: (pid 23701) 84s
run: postgres-exporter: (pid 24240) 38s; run: log: (pid 23995) 54s
run: postgresql: (pid 23185) 148s; run: log: (pid 23260) 147s
run: prometheus: (pid 24115) 40s; run: log: (pid 23899) 66s
run: redis: (pid 22967) 155s; run: log: (pid 23009) 154s
run: redis-exporter: (pid 24103) 40s; run: log: (pid 23820) 72s
run: sidekiq: (pid 23477) 111s; run: log: (pid 23494) 110s
run: unicorn: (pid 23420) 117s; run: log: (pid 23474) 114s
登录http://39.106.96.125:65535测试root/root@123


nginx : web入口
database（postgresql，mysql） （gitlab repository issue，merge request等，用户（权限））
redis 缓存，负责分发任务
sideiq：后台任务，主要负责发送电子邮件，任务需要来自redis
unicorn： 包含gitlab 主进程
gitlab-shell 用于ssh交互
gitlab-workhorse：反向代理服务器，可以处理与unicorn 无关的请求，处理git pull／push请求，处理到unicorn 的连接
gitaly 后台服务，用于处理GitLab发出的所有git调用
systemctl enable gitlab-runsvdir.service
systemctl start gitlab-runsvdir.service



gitlab-ctl start #启动全部服务
gitlab-ctl restart #重启全部服务
gitlab-ctl stop #停止全部服务
gitlab-ctl restart nginx #重启单个服务
gitlab-ctl status #查看全部组件的状态
gitlab-ctl show-config #验证配置文件
gitlab-ctl uninstall #删除gitlab(保留数据）
gitlab-ctl cleanse #删除所有数据，重新开始
gitlab-ctl tail <svc_name>  #查看服务的日志
gitlab-rails console production #进入控制台 ，可以修改root 的密码

```

### 使用

```
注册：fengk fengk@126.com fengk@123

创建project===》pj-test

获取project地址：http://39.106.96.125:65535/fengkai/pj-test.git
192.168.106.117安装git

```

### gitlab-runner 使用
```
192.168.106.117
[root@managementa ~]# sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
[root@managementa ~]# chmod +x /usr/local/bin/gitlab-runner
[root@managementa ~]# gitlab-runner --version
Version:      12.4.1
Git revision: 05161b14
Git branch:   12-4-stable
GO version:   go1.10.8
Built:        2019-10-28T12:49:57+0000
OS/Arch:      linux/amd64
[root@managementa ~]# useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
可以指定root帐号，此时会减少一些权限的麻烦gitlab-runner install --user=root --working-directory=/home/gitlab-runner
[root@managementa ~]# gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
Runtime platform                                    arch=amd64 os=linux pid=5458 revision=05161b14 version=12.4.1
[root@managementa ~]# gitlab-runner start
Runtime platform                                    arch=amd64 os=linux pid=5719 revision=05161b14 version=12.4.1

打开 gitlab 项目 -> 设置 -> CI / CD -> Runners 设置

[root@managementa ~]# gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=10714 revision=05161b14 version=12.4.1
Running in system-mode.                            
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://39.106.96.125:65535/
Please enter the gitlab-ci token for this runner:
hoz1f8fL3ZoMyyYmbLc5
Please enter the gitlab-ci description for this runner:
[managementa]: gitlab runner ci 测试
Please enter the gitlab-ci tags for this runner (comma separated):
pj-test-tag
Registering runner... succeeded                     runner=hoz1f8fL
Please enter the executor: docker-ssh+machine, docker-ssh, parallels, shell, ssh, virtualbox, docker+machine, kubernetes, custom, docker:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
[root@managementa gitlab-runner]# gitlab-runner list

```

### 编写 .gitlab-ci.yml 集成

```
[root@managementa gitlab-runner]# git clone http://39.106.96.125:65535/fengkai/pj-test.
[root@managementa gitlab-runner]# cd pj-test/
[root@managementa pj-test]# git remote -v
origin  http://39.106.96.125:65535/fengkai/pj-test.git (fetch)
origin  http://39.106.96.125:65535/fengkai/pj-test.git (push)
[root@managementa pj-test]# git config --global user.email "fengk@126.com"
[root@managementa pj-test]# git config --global user.name "fengk"
[root@managementa pj-test]# git add *
[root@managementa pj-test]# git commit -m "tomcat" 
[root@managementa pj-test]# git push  用户名要填入邮件名

[root@managementa builds]# chmod 666 /var/run/docker.sock 其他用户可以访问docker
[root@managementa pj-test]# cat Dockerfile 
FROM 192.168.106.117/library/tomcat:8.5.35
ADD webCache.war /usr/local/tomcat/webapps/
[root@managementa pj-test]# cat .gitlab-ci.yml 
stages:
 - deploy
job-build-image:
 stage: deploy
 allow_failure: true
 script:
  - /usr/local/docker/docker build -t 192.168.106.117/library/tomcat-webcache:0.01 -f Dockerfile .
  - echo $?
  - /usr/local/docker/docker push 192.168.106.117/library/tomcat-webcache:0.01
  - echo $?
 only:
  - master
 tags:
  - hch-test 

[root@managementa pj-test]# 

[root@managementa pj-test]# ls
Dockerfile  README.md  webCache.war
```