---
title: 时间同步与tomcat自启动
tags:
  - tool
categories:
  - linux
date: 2019-02-13 05:00:00
---
> 时间同步与tomcat自启动
<!-- more -->

## 时间同步
### 修改配置
```
vi /etc/chrony.conf
server 10.24.70.72 iburst
```
### 重启
```
systemctl stop chronyd
systemctl start chronyd
```
### 查看
```
chronyc sources
```
## tomcat自启动
### 添加变量到setenv.sh（没有的话在bin目录新建一个）中,catalina.sh会调用
```
vi /usr/local/apache-tomcat-8.5.35/bin/setenv.sh
JAVA_HOME=/opt/jdk1.8.0_144
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
CATALINA_PID="$CATALINA_BASE/tomcat8_hcpt.pid"
```
### 设置启动脚本
```
vi /usr/lib/systemd/system/tomcat8_hcpt.service
#/usr/lib/systemd/system/tomcat8_hcpt.service
[Unit]
Description=Tomcat8
After=syslog.target network.target remote-fs.target nss-lookup.target
[Service]
Type=forking
PIDFile=/usr/local/apache-tomcat-8.5.35/tomcat8_hcpt.pid
ExecStart=/usr/local/apache-tomcat-8.5.35/bin/startup.sh
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```
### 启动
```
systemctl enable tomcat8_hcpt
systemctl start tomcat8_hcpt
```

## centos7+tomcat9自启动
### 添加变量到setclasspath.sh中,catalina.sh会调用
```
vi /usr/local/tomcat9/bin/setclasspath.sh
# -----------------------------------------------------------------------------
#  Set JAVA_HOME or JRE_HOME if not already set, ensure any provided settings
#  are valid and consistent with the selected start-up options and set up the
#  endorsed directory.
# -----------------------------------------------------------------------------
export JAVA_HOME=/usr/local/jdk
export JRE_HOME=/usr/local/jdk/jre
```
### 设置启动脚本
```
vi /usr/lib/systemd/system/tomcat9.service
#/usr/lib/systemd/system/tomcat9.service
[Unit]
Description=Tomcat9
After=syslog.target network.target remote-fs.target nss-lookup.target
[Service]
Type=oneshot
PIDFile=/usr/local/tomcat9/tomcat.pid
ExecStart=/usr/local/tomcat9/bin/startup.sh
ExecStop=/usr/local/tomcat9/bin/shutdown.sh
ExecReload=/bin/kill -s HUP $MAINPID
RemainAfterExit=yes
[Install]
WantedBy=multi-user.target
```
### 启动
```
systemctl enable tomcat9
systemctl start tomcat9
```