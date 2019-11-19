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