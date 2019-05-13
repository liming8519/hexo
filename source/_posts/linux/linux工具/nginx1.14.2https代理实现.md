---
title:  nginx1.14.2https代理实现
tags:
- nginx 
categories: 
- linux 
date: 2019-05-09 02:00:00
---
> nginx1.14.2https代理实现
<!-- more -->

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

# ./configure \
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

#vi /usr/lib/systemd/system/nginx.service
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
#systemctl start nginx
#systemctl enable nginx

```


## 手动配置ssl证书
```
cd /etc/nginx
openssl genrsa -des3 -out cert.key 1024
密码：123456
openssl req -new -key cert.key -out cert.csr
Enter pass phrase for cert.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:hebei
Locality Name (eg, city) [Default City]:shijiazhuang
Organization Name (eg, company) [Default Company Ltd]:hbfec
Organizational Unit Name (eg, section) []:soft
Common Name (eg, your name or your server's hostname) []:ythserver
Email Address []:yth@hbfec.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:123456
An optional company name []:hbfec

cp cert.key cert.key.bak
openssl rsa -in cert.key.bak -out cert.key
openssl x509 -req -days 365 -in cert.csr -signkey cert.key -out cert.crt
```


## 配置
```
# vi /etc/nginx/nginx.conf
    upstream baidu{
        server 103.235.46.39;
    }
    server {
        listen       443 ssl;
        server_name  192.168.106.219;
        #ssl证书文件位置(常见证书文件格式为：crt/pem)
        ssl_certificate      cert.crt;
        #ssl证书key位置
        ssl_certificate_key  cert.key;
        #ssl配置参数（选择性配置）
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        #数字签名，此处使用MD5
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        #编码格式
        charset utf-8;
        #代理配置参数
        proxy_connect_timeout 180;
        proxy_send_timeout 180;
        proxy_read_timeout 180;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarder-For $remote_addr;
 
        #反向代理的路径（和upstream绑定），location 后面设置映射的路径
        location / {
            proxy_pass https://baidu;
        } 
 

    
        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status           on;
            access_log            on;
            auth_basic            "NginxStatus";
            auth_basic_user_file  conf/htpasswd;
        }
    
    }
```