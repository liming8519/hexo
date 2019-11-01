---
title:  openvswitch2.7 安装
tags:
- 虚拟化
categories: 
- linux 
date: 2019-10-30 02:00:00
---
> openvswitch2.7 安装
<!-- more -->

## openvswitch安装

  
```
1、下载ovs 2.7.0代码
$ git clone https://github.com/openvswitch/ovs.git
$ wget http://www.openvswitch.org//releases/openvswitch-2.7.0.tar.gz

2、配置环境预准备（必须使用root权限）

./boot.sh
曾经出现的问题：
[zhou@localhost ovs]$ ./boot.sh
./boot.sh: line 2: autoreconf: command not found
解决方法：
[root@localhost ovs]# yum install autoconf automake libtool

3、配置环境（必须使用root权限）

#./configure

4、编译（必须使用root权限）

# make

5、检查是否能通过ovs自带的单元测试用例

$ make check

6、安装（必须使用root权限）

# make install

7、检查安装是否成功？（必须使用root权限）

# /sbin/lsmod | grep openvswitch



开始使用（必须使用root权限）

# export PATH=$PATH:/usr/local/share/openvswitch/scripts
# ovs-ctl start

验证工作（必须使用root权限）

# ovs-vsctl add-br br0
# ovs-vsctl add-port br0 eth0
# ovs-vsctl add-port br0 vif1.0
```




