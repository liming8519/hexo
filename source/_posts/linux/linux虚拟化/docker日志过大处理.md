#  docker日志过大处理

---
title:  docker日志过大处理
tags:
- 虚拟化
categories: 
- linux 
date: 2019-11-19 02:00:00
---
> docker日志过大处理
<!-- more -->

### 设置一个容器服务的日志大小上限

```
nginx: 
  image: nginx:1.12.1 
  restart: always 
  logging: 
    driver: “json-file” 
    options: 
      max-size: “5g” 
```

### 全局设置

```
# vim /etc/docker/daemon.json

{
  "registry-mirrors": ["http://f613ce8f.m.daocloud.io"],
  "log-driver":"json-file",
  "log-opts": {"max-size":"500m", "max-file":"3"}
}
```