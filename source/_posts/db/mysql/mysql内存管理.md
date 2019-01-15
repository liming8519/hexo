---
title: Mysql 内存管理
tags:
  - mysql
categories:
  - db
date: 2019-01-15 00:00:00
---
> 分析mysql内存结构
<!-- more -->
## 内存结构图
![mysql内存结构描述][1]

1. 全局缓存包括
```
> global buffer(全局内存分配总和) =
> innodb_buffer_pool_size --InnoDB高速缓冲，行数据、索引缓冲，以及事务锁、自适应哈希等
> +innodb_additional_mem_pool_size  --InnoDB数据字典额外内存，缓存所有表数据字典
> +innodb_log_buffer_size -- InnoDB REDO日志缓冲，提高REDO日志写入效率
> +key_buffer_size -- MyISAM表索引高速缓冲，提高MyISAM表索引读写效率
> +query_cache_size --查询高速缓存，缓存查询结果，提高反复查询返回效率
> +thread_cache_size --Thread_Cache 中存放的最大连接线程数
> +table_cahce -- 表空间文件描述符缓存，提高数据表打开效率
> +table_definition_cache -- 表定义文件描述符缓存，提高数据表打开效率
```
2. 会话缓存包括
```
> total_thread_buffers= max_connections*(
>  read_buffer_size-- 顺序读缓冲，提高顺序读效率
> +read_rnd_buffer_size-- 随机读缓冲，提高随机读效率
> +sort_buffer_size-- 排序缓冲，提高排序效率
> +join_buffer_size-- 表连接缓冲，提高表连接效率
> +binlog_cache_size-- 二进制日志缓冲，提高二进制日志写入效率
> +tmp_table_size -- 内存临时表，提高临时表存储效率
> +thread_stack-- 线程堆栈，暂时寄存SQL语句/存储过程
> +thread_cache_size-- 线程缓存，降低多次反复打开线程开销，模拟连接池
> )
```
## 查看Mysql全局占用内存大小
``` mysql
mysql>select (@@innodb_buffer_pool_size+@@innodb_log_buffer_size+@@key_buffer_size)/1024/1024 as g_buffer;

```
## performance_schema占用多少内存
```mysql
mysql>SELECT SUBSTRING_INDEX(event_name,'/',2) AS  
         code_area, sys.format_bytes(SUM(current_alloc))  
         AS current_alloc  
         FROM sys.x$memory_global_by_current_bytes  
         GROUP BY SUBSTRING_INDEX(event_name,'/',2)  
         ORDER BY SUM(current_alloc) DESC; 
```
## 每个线程占用多少内存 
``` mysql
mysql>SELECT ( ( @@read_buffer_size  
   + @@read_rnd_buffer_size  
   + @@sort_buffer_size  
   + @@join_buffer_size  
   + @@binlog_cache_size  
   + @@thread_stack  
   + @@max_allowed_packet  
   + @@net_buffer_length )  
   ) / (1024*1024) AS MEMORY_MB; 
   
```


  
  
  
  


  
  
  
  
  [1]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/mysql%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.jpg