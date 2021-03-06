---
title: nginx1.14.2 安装配置文档
tags:
  - tool
categories:
  - linux
date: 2019-02-13 01:00:00
---
> nginx安装配置文档
<!-- more -->


## 配置本地yum源
```
#cd /etc/yum.repos.d/
#tar -cvf all.repo.tar *
#rm -fr *.repo
#vi cdiso.repo
[centos]
name=centos
baseurl=file:///root/c7
gpgcheck=0
enabled=1

#把iso挂载到/root/c7目录下
#mkdir /root/c7
#mount -o loop 系统iso文件.iso /root/c7
```
## 基本环境配置
```
#yum install gcc pcre-devel zlib-devel openssl-devel
#systemctl stop firewalld
#systemctl disable firewalld
#groupadd nginx
#useradd nginx -g nginx -s /sbin/nologin -M
#vi /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
```
## 安装
```
#./configure \
--prefix=/usr/local/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_gzip_static_module \
--http-log-path=/var/log/nginx/access.log \
--with-http_stub_status_module \
--with-openssl=
#make
#make install
```
## 启动脚本
- 编辑文件
```
#vi /usr/lib/systemd/system/nginx.service
```
- 文件内容如下
```
#/usr/lib/systemd/system/nginx.service
[Unit]
Description=nginx
After=network.target
[Service]
Type=forking
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/usr/sbin/nginx -s stop
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```
## 配置
- 编辑配置文件
```
vi /etc/nginx/nginx.conf
```
- 编辑文件内容如下
```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log;
pid        /var/log/nginx/nginx.pid;

events {
    worker_connections  65535;
    use epoll;
    multi_accept on;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    charset utf-8;

    tcp_nodelay    on;
    
    server_tokens off;
    
    open_file_cache max=65535 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 1;
    
    client_header_buffer_size 4k;
    large_client_header_buffers 4 32k;
    client_header_timeout 15;
    client_body_timeout 15;
    client_max_body_size 8m;
    
    client_body_temp_path /usr/local/nginx/body_temp 1 2;
    
    reset_timedout_connection on;
    send_timeout 30;
    
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;
    
    upstream hch80{
        server 10.24.70.81:80;
    }
    upstream hch18080{
        server 10.24.70.82:18080;
    }
    server {
        listen       80;
        server_name  10.24.70.74;

        location / {
            proxy_pass http://hch80;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_connect_timeout 90;
            proxy_buffer_size 4k;
            proxy_buffers 4 32k;
            proxy_busy_buffers_size 64k;
            proxy_temp_file_write_size 64k;
            proxy_read_timeout 90;
            proxy_send_timeout 90;
            proxy_temp_path /usr/local/nginx/proxy_temp80 1 2;
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
       
    }
    server {
        listen       18080;
        server_name  10.24.70.74;
        location / {
            proxy_pass http://hch18080;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_connect_timeout 90;
            proxy_buffer_size 4k;
            proxy_buffers 4 32k;
            proxy_busy_buffers_size 64k;
            proxy_temp_file_write_size 64k;
            proxy_read_timeout 90;
            proxy_send_timeout 90;
            proxy_temp_path /usr/local/nginx/proxy_temp18080 1 2;
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## 启动
```
systemctl start nginx
systemctl enable nginx
```