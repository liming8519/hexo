
---
title: BI相关工具总结
tags:
  - tool
categories:
  - linux
date: 2019-09-25 01:00:00
---
> BI相关工具总结
<!-- more -->



## BI相关工具总结

### 数据可视化
#### superset
略
#### metabase
```

	[root@controller fk]# wget https://downloads.metabase.com/v0.33.3/metabase.jar
	[root@controller fk]# java -jar metabase.jar 
```
#### redash

```

	[root@VM_38_115_centos redash]# vi docker-compose.production.yml
	# This is an example configuration for Docker Compose. Make sure to atleast update
	# the cookie secret & postgres database password.
	#
	# Some other recommendations:
	# 1. To persist Postgres data, assign it a volume host location.
	# 2. Split the worker service to adhoc workers and scheduled queries workers.
	version: '2'
	services:
	  server:
	    image: redash/redash:latest
	    command: server
	    depends_on:
	      - postgres
	      - redis
	    ports:
	      - "5000:5000"
	    environment:
	      PYTHONUNBUFFERED: 0
	      REDASH_LOG_LEVEL: "INFO"
	      REDASH_REDIS_URL: "redis://redis:6379/0"
	      REDASH_DATABASE_URL: "postgresql://postgres@postgres/postgres"
	      REDASH_COOKIE_SECRET: "Q422k6vaXUk8"
	      REDASH_WEB_WORKERS: 4      
	    restart: always
	  worker:
	    image: redash/redash:latest
	    command: scheduler
	    environment:
	      PYTHONUNBUFFERED: 0
	      REDASH_LOG_LEVEL: "INFO"
	      REDASH_REDIS_URL: "redis://redis:6379/0"
	      REDASH_DATABASE_URL: "postgresql://postgres@postgres/postgres"
	      QUEUES: "queries,scheduled_queries,celery"
	      WORKERS_COUNT: 2
	    restart: always
	  redis:
	    image: redis:3.0-alpine
	    ports:
	    - "6379:6379"
	    volumes: 
	      - ./data/redis_data:/data
	    restart: always
	  postgres:
	    image: postgres:9.5.6-alpine
	    ports:
	    - "5432:5432"
	    volumes:
	      - ./data/postgresql_data:/var/lib/postgresql/data
	    restart: always
	  nginx:
	    image: redash/nginx:latest
	    ports:
	      - "88:80"
	    depends_on:
	      - server
	    links:
	      - server:redash
	    restart: always
	[root@VM_38_115_centos redash]# docker-compose -f docker-compose.production.yml run --rm server create_db
	[root@VM_38_115_centos redash]# docker-compose -f docker-compose.production.yml up
```

#### zeppelin
大数据
#### sqlpad

略

#### cboard

国产的
https://peter_zhang921.gitee.io/cboard_docsify/#/


#### FineBI

finebi   https://help.finebi.com/
finereport   https://help.finereport.com/

试用码：dbe8992a-085c130cf-2278-d0857c2db051

### web报表技术(java)
* jfreechart
* cewolf
* jcharts
* ireport
* jasperreports
* jfreereport
* openreport
* birt