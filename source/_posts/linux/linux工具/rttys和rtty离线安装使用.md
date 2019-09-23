---
title:  rttys和rtty离线安装使用
tags:
- tool
categories: 
- linux 
date: 2019-09-23 02:00:00
---
> rttys和rtty离线安装使用
<!-- more -->

## rttys离线安装说明

> https://github.com/zhaojh329/rttys/releases下载合适的rttys


```

	[root@sdw3 ~]# tar -xzvf rttys-linux-amd64.tar.gz 

```

### 查看支持哪些命令行参数

```

	./rttys -h
	Usage of ./rttys:
	  -addr string
	        address to listen (default ":5912")
	  -conf string
	        config file to load (default "./rttys.conf")
	  -gen-token
	        generate token
	  -ssl-cert string
	        certFile Path (default "./rttys.crt")
	  -ssl-key string
	        keyFile Path (default "./rttys.key")
	  -token string
	        token to use
```

### 以root用户运行(使用系统用户名和密码)

```

	sudo ./rttys

```

### 以普通用户运行(用户名和密码来自配置文件)
```
	./rttys
```
### 如何在后台运行模式下查看日志
```
	cat /var/log/rttys.log
```
### 认证
```

	./rttys -gen-token
	Please set a password:******
	Your token is: 34762d07637276694b938d23f10d7164
	
	./rttys -token 34762d07637276694b938d23f10d7164
```


### 查看

> 帐号密码为服务器帐号密码

```
https://192.168.106.237:5912
```







## rtty离线安装使用

### 客户端依赖

* libev 高性能的事件循环库
* libuwsc 一个轻量的针对嵌入式Linux的基于libev的WebSocket客户端C库。
* mbedtls(polarssl)、CyaSSl(wolfssl)或者openssl 如果你需要支持SSL

>离线安装例子
>libev：地址=》https://src.fedoraproject.org/repo/pkgs/libev/libev-4.27.tar.gz/sha512/18fbac15c3a24b2efcd547d98d423fe59a1684cd3afe7ff25a3da54d8df3e11f351df455657d830df93366853f74d584f6e47a7c9ffaba84aa586957bf39ea82/libev-4.27.tar.gz：./configure && make && makeinstall
>libywsc: 再下面的sh中通过git下载后编译安装
>openssl：使用yum安装，搭建本地yum源即可安装

### 安装rtty

```
wget -qO- https://raw.githubusercontent.com/zhaojh329/rtty/master/tools/install.sh | sudo bash
```

>install.sh说明




```

	#!/bin/bash
	LSB_ID=
	INSTALL=
	REMOVE=
	UPDATE=
	PKG_LIBEV="libev-dev"
	PKG_SSL="libssl-dev"
	show_distribution() {

        local pretty_name=""
        if [ -f /etc/os-release ];
        then
                . /etc/os-release
                pretty_name="$PRETTY_NAME"
                LSB_ID="$(echo "$ID" | tr '[:upper:]' '[:lower:]')"
        elif [ -f /etc/redhat-release ];
        then
                pretty_name=$(cat /etc/redhat-release)
                LSB_ID="$(echo "$pretty_name" | tr '[:upper:]' '[:lower:]')"
                echo "$LSB_ID" | grep centos > /dev/null && LSB_ID=centos
        fi
        LSB_ID=$(echo "$LSB_ID" | tr '[:upper:]' '[:lower:]')
        echo "Platform: $pretty_name"
	}
	check_cmd() {
        which $1 > /dev/null 2>&1
	}
	detect_pkg_tool() {
        check_cmd apt && {
                UPDATE="apt update -q"
                INSTALL="apt install -y"
                REMOVE="apt remove -y"
                return 0
        }
        check_cmd apt-get && {
                UPDATE="apt-get update -q"
                INSTALL="apt-get install -y"
                REMOVE="apt-get remove -y"
                return 0
        }
        check_cmd yum && {
                UPDATE="yum update -yq"
                INSTALL="yum install -y"
                REMOVE="yum "
                PKG_LIBEV="libev-devel"
                PKG_SSL="openssl-devel"
                return 0
        }
        check_cmd pacman && {
                UPDATE="pacman -Sy --noprogressbar"
                INSTALL="pacman -S --noconfirm --noprogressbar"
                REMOVE="pacman -R --noconfirm --noprogressbar"
                PKG_LIBEV="libev"
                PKG_SSL="openssl"
                return 0
        }
        return 1
	}
	check_tool() {
        local tool=$1

        check_cmd $tool || $INSTALL $tool
	}
	show_distribution
	detect_pkg_tool || {
        echo "Your platform is not supported by this installer script."
        exit 1
	}
	#$UPDATE
	[ "$LSB_ID" = "centos" ] && $INSTALL epel-release
	check_tool pkg-config
	check_tool gcc
	check_tool make
	check_tool cmake
	check_tool git
	$INSTALL $PKG_LIBEV
	$INSTALL $PKG_SSL
	rm -rf /tmp/rtty-build
	mkdir /tmp/rtty-build
	pushd /tmp/rtty-build
	git clone --recursive https://github.com/zhaojh329/libuwsc.git || {
        echo "Clone libuwsc failed"
        exit 1
	}
	sleep 2
	git clone https://github.com/zhaojh329/rtty.git || {
        echo "Clone rtty failed"
        exit 1
	}
	# libuwsc
	rm -f /usr/local/lib/libuwsc.*
	cd libuwsc && cmake . && make install && cd -
	[ $? -eq 0 ] || exit 1
	# rtty
	cd rtty && cmake . && make install
	[ $? -eq 0 ] || exit 1
	popd
	rm -rf /tmp/rtty-build
	ldconfig
	case "$LSB_ID" in
        centos|arch)
                echo "/usr/local/lib" > /etc/ld.so.conf.d/rtty
                ldconfig -f /etc/ld.so.conf.d/rtty
        ;;
	esac
	rtty -V
```


### 查看命令行选项rtty

```

	Usage: rtty [option]
	  -i ifname    # Network interface name - Using the MAC address of
                      the interface as the device ID
	  -I id        # Set an ID for the device(Maximum 63 bytes, valid character:letters
                      and numbers and underlines and short lines) - If set,
                      it will cover the MAC address(if you have specify the ifname)
	  -h host      # Server host
	  -p port      # Server port(Default is 5912)
	  -a           # Auto reconnect to the server
	  -v           # verbose
	  -d           # Adding a description to the device(Maximum 126 bytes)
	  -s           # SSL on
	  -k keepalive # keep alive in seconds for this client. Defaults to 5
	  -V           # Show version
	  -D           # Run in the background
	  -t token     # Authorization token
```

### 运行RTTY(将下面的参数替换为你自己的参数)rtty

```
sudo rtty -I 'My-device-ID' -h 'your-server' -p 5912 -a -v -s -d 'My Device Description'
实例，不能用root登录可以用其他帐号登录
rtty -I "237" -h 192.168.106.237 -p 5912 -s
```
### 如果你的rttys配置了一个token，请加上如下参数（将下面的token替换为你自己生成的）rtty
```
-t 34762d07637276694b938d23f10d7164
```

## 如何让root能够登录

```

加入如下配置

	[root@sdw3 ~]# vi /etc/securetty

	pts/0
	pts/1
	pts/2
	[root@sdw3 ~]# cd /etc/pam.d
	[root@sdw3 pam.d]# cat login 
	#%PAM-1.0
	auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so
	auth       substack     system-auth
	auth       include      postlogin
	account    required     pam_nologin.so
	account    include      system-auth
	password   include      system-auth
	# pam_selinux.so close should be the first session rule
	session    required     pam_selinux.so close
	session    required     pam_loginuid.so
	session    optional     pam_console.so
	# pam_selinux.so open should only be followed by sessions to be executed in the user context
	session    required     pam_selinux.so open
	session    required     pam_namespace.so
	session    optional     pam_keyinit.so force revoke
	session    include      system-auth
	session    include      postlogin
	-session   optional     pam_ck_connector.so
	[root@sdw3 pam.d]# cat sshd 
	#%PAM-1.0
	auth       required     pam_sepermit.so
	auth       substack     password-auth
	auth       include      postlogin
	# Used with polkit to reauthorize users in remote sessions
	-auth      optional     pam_reauthorize.so prepare
	account    required     pam_nologin.so
	account    include      password-auth
	password   include      password-auth
	# pam_selinux.so close should be the first session rule
	session    required     pam_selinux.so close
	session    required     pam_loginuid.so
	# pam_selinux.so open should only be followed by sessions to be executed in the user context
	session    required     pam_selinux.so open env_params
	session    required     pam_namespace.so
	session    optional     pam_keyinit.so force revoke
	session    include      password-auth
	session    include      postlogin
	# Used with polkit to reauthorize users in remote sessions
	-session   optional     pam_reauthorize.so prepare
	[root@sdw3 pam.d]# cat system-auth
	#%PAM-1.0
	# This file is auto-generated.
	# User changes will be destroyed the next time authconfig is run.
	auth        required      pam_env.so
	auth        sufficient    pam_unix.so nullok try_first_pass
	auth        requisite     pam_succeed_if.so uid <= 1000 quiet_success
	auth        required      pam_deny.so
	
	account     required      pam_unix.so
	account     sufficient    pam_localuser.so
	account     sufficient    pam_succeed_if.so uid < 1000 quiet
	account     required      pam_permit.so
	
	password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
	password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
	password    required      pam_deny.so
	
	session     optional      pam_keyinit.so revoke
	session     required      pam_limits.so
	-session     optional      pam_systemd.so
	session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
	session     required      pam_unix.so
	[root@sdw3 pam.d]# 
	修改==》
	auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
	为==》
	auth        requisite     pam_succeed_if.so uid <= 1000 quiet_success

```