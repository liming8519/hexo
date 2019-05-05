

---
title: fastdfs-nginx-module安装
tags:
  - tool
categories:
  - linux
date: 2019-05-06 01:00:00
---
> fastdfs-nginx-module安装
<!-- more -->


## fastdfs-nginx-module安装

### 初始化

```
yum -y  install pcre-devel opensslopenssl-devel zlib-devel zlib
```

### 下载fastdfs-nginx-module
```
wget https://github.com/happyfish100/fastdfs-nginx-module/archive/master.zip
```

### nginx使用版本1.14.2
###解压
```
unzip master.zip
tar -xzvf nginx-1.14.2.tar.gz
mv nginx-1.14.2 /usr/local
cd /usr/local/nginx-1.14.2
./configure --add-module=/root/fastdfs-nginx-module-master/src/ --prefix=/usr/local/nginx-1.14.2 --user=root --group=root --with-http_gzip_static_module --with-http_gunzip_module
#报错解决
vi /root/fastdfs-nginx-module-master/src/config
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"

make
make install

vi /etc/fdfs/mod_fastdfs.conf 
connect_timeout=10
tracker_server=127.0.0.1:22122
url_have_group_name=true
store_path0=/home/filedata/storage/data
group_name=group1
base_path=/home/filedata/storage/info
#include http.conf

cd fastdfs-5.11
cd conf
cp http.conf mime.types  /etc/fdfs/

mkdir /usr/local/nginx-1.14.2/logs
vi /usr/local/nginx-1.14.2/conf/nginx.conf
在http部分中添加配置信息如下：
#修改第一行
user root;
#然后加入
events {
  worker_connections  1024;  ## Default: 1024
}
http
{
       server {
            listen       8888;
            server_name  localhost;
            location ~/group[0-9]/ {
                ngx_fastdfs_module;
            }
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
            root   html;
            }
        }
}

```