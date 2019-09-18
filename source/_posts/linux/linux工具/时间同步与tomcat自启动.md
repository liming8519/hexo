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
### 修改配置(同步其他服务器时间同时作为时间服务器)
```
[root@localhost ~]# cat /etc/chrony.conf 
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst

server 20.120.16.37 iburst

# Ignore stratum in source selection.
stratumweight 0
# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift
# Enable kernel RTC synchronization.
rtcsync
# In first three updates step the system clock instead of slew
# if the adjustment is larger than 10 seconds.
makestep 10 3
# Allow NTP client access from local network.

allow 10.28/16

# Listen for commands only on localhost.
bindcmdaddress 127.0.0.1
bindcmdaddress ::1
# Serve time even if not synchronized to any NTP server.

local stratum 10

keyfile /etc/chrony.keys
# Specify the key used as password for chronyc.
commandkey 1
# Generate command key if missing.
generatecommandkey
# Disable logging of client accesses.
noclientlog
# Send a message to syslog if a clock adjustment is larger than 0.5 seconds.
logchange 0.5
logdir /var/log/chrony
#log measurements statistics tracking
[root@localhost ~]# 
```
### 修改配置(同步其他服务器的时间)
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
### 如果同步时间出现不同步时间的问题请关闭selinux试试


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