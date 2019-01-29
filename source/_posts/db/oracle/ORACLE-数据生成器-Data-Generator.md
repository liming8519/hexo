---
title: ORACLE 数据生成器 Data Generator
tags:
  - oracle
categories:
  - db
date: 2019-01-29 02:00:00

---
>ORACLE生成测试数据

<!-- more -->


- 在psql的工具菜单下找到数据生成器：Data Generator
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190129/1548743024766.png)

- psql有一些帮助信息
![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190129/1548743314201.png)

- 例子
1. SQL(sys_guid())：guid
2. List(1,2,3,4,5,6,7,8,9,10,11,12)：从list中随机选数据
3. SQL(sysdate-dbms_random.value(1,180))：半年内时间生成



