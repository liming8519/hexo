---
title: oracle 导入导出文档
tags:
  - oracle
categories:
  - db
date: 2019-01-16 02:00:00

---

> oracle 数据泵导入导出

<!-- more -->


##	导出
- 首先建立directory，directory是存放备份文件的路径（导入导出都要建立,对应的C:/exp文件夹如果不存在，需要手工建立）
``` sql
SQL> connect sys/123456 as sysdba
SQL> create or replace directory expdir as 'C:/exp';
SQL> grant read,write on directory expdir to public;
#查看directory是否创建成功
SQL> select * from dba_directories;
SQL> commit;
```
- 使用expdp命令导出某个用户的数据
```
expdp kliu_voi5/1234@orcl schemas=kliu_voi5 dumpfile=kliu_voi5.dmp directory=expdir
```


> 其中kliu_voi5/1234是用户名密码，orcl是oracle的sid。该命令完成后在c:\exp下会有kilu_voi5.dmp文件


- 如果只是要导出某些表，可以使用include
```
expdp kliu_voi5/1234@orcl schemas=kliu_voi5 dumpfile=kliu_voi5.dmp directory=expdir include=table:\"like \'CT%\'\"
```

##	导入
- 首先建立directory
```
SQL> connect sys/123456 as sysdba
SQL> create or replace directory db_bak as 'd:/db_bak';
SQL> grant read,write on directory db_bak to public;
SQL> select * from dab_directories;
SQL> commit;
```

- 如果是在新的oracle数据库里，需用重建同样的用户
```
SQL> connect sys/123456 as sysdba
SQL> create user kliu_voi5 identified by "1234";
SQL> grant dba to kliu_voi5;
SQL> commit;
```

- 使用impdp命令导入某个用户的数据
```
impdp kliu_voi5/1234@orcl directory=db_bak dumpfile=kliu_voi5.dmp full=y
```

##	帮助使用方法
- 导出命令打印帮助信息
```
expdp help=y
```
