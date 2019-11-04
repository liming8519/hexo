---
title:  noVnc 安装
tags:
- tool
categories: 
- linux 
date: 2019-11-04 02:00:00
---
> noVnc 安装
<!-- more -->

## noVnc 安装

```

#!/bin/bash

# stop selinux and iptables
setenforce 0
systemctl stop firewalld
systemctl disable firewalld

# install vncserver and git
yum install -y epel*
yum install tigervnc-server git -y
vncserver :1
# 此时会提示输入密码

# download noVNC
git clone git://github.com/kanaka/noVNC

# create secure connection
cd ./noVNC/utils/
openssl req -new -x509 -days 365 -nodes -out self.pem -keyout self.pem

# run noVNC
cd ../
./utils/launch.sh --vnc localhost:5901

# running
```