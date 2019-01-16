---
title: sqlserver 文件存储笔记
tags:
  - sqlserver
categories:
  - db
date: 2019-01-16 01:00:00

---

> sqlserver 笔记，文件存储相关
<!-- more -->

## page页
- 每个页面8KB，连续的8个页面称为一个区extents。
- 统一区：8个页面均为一个对象所有。
- 混合区： 8个页面最多可被8个对象共享。
## %%lockres%% %%physloc%%
- lockres ： 锁
- physloc ： 物理位置，64位的计算方式是，如0x0001000001000000则0x0000000100000100，前4为为槽号0x0000，中间4位为文件号0x0001，最后8位0x00000100是页号。
- 显示： 文件编号：页编号：页内位置

## GAM页
- 全局分配映射(Global Allocation Map)，包含页头和一些其它开销外，还有8 000字节或者说64 000bit位可用，每个bit位代表一个区（8个page），0表示已使用，1表示自由区。

## SGAM页

- 共享全局分区，类似GAM一个bit表示一个区，不同的是，他的1表示混合区且有可用空间；0表示未使用或无可用空间。**用来跟踪区的分配情况，描述哪些区是混合区并且至少有一个空闲的数据页**。

## 系统page页
- 个GAM页面出现在第一个GAM页面(页码为2)以后的每511230个页面中（大约4G空间后），并且下一个SGAM页面出现在第一个SGAM页面(页码为3)以后的每511 230个页面中。
- 每一个数据库文件的页码为0的页面是文件头页面，并且每个文件仅有一页（文件头页面，页码为0）。
- 页码0是头文件页，页码1是页面自由空间页(Page Free Space，PFS)。


|第0页      |第1页     |第2页    |第3页    |第4页    |第5页    |第6页    |第7页  
|-----------|----------|---------|---------|---------|---------|---------|-------
|m_type=15	|m_type=11 |m_type=8 |m_type=9 |m_type=0 |m_type=0 |m_type=16|	m_type=17
|头文件页	|PFS页     |GAM页    |SGAM页   |保留页   |保留页   |DCM页    |	BCM页


- 除了第9页为数据库的BOOT页以外，从第8页到第173页为SQLServer2008内部系统表的相关存储信息，然后从第174页到第279页为未分配页面。因为第一页从0开始，所以刚好280页，即和我们看到的数据库数据文件的大小完全相等。
- 见前文的运算，数据库大小：2.18 MB (2,293,760 字节)=2,293,760b/8kb=280个页面=35个区。


|第8页    |第9页    |第10页~第173页|第174~279页|
|---------|---------|----------|-------        |
|m_type=1 |m_type=13|m_type in (1,2,10)|N/A     |
|Data页   |Boot页   |主要为内部系统表相关信息|未分配|

## PFS

- 用来跟踪页分配级别，存储当前数据文件里所有页分配及可用空间的信息，每一个数据文件的第2个数据页都是PFS，页号为1 。该页面中，每一个字节描述后面每一个数据页是否还有空间可以写记录，也就是一个PFS页是8k，约有8k个字节可以描述后续每个页面的使用情况，也就是一个PFS页，可以描述8k个数据页的使用情况，这就意味着单个PFS页能够存储约64M数据页的可用空间情况。所以，大约每隔64Mb，就会有一个新的PFS页。


|位置|含义|
|----|----|
|bit 0-2|0x00 is empty;0x01 is 1 to 50% full;0x02 is 51 to 80% full;0x03 is 81 to 95% full;0x04 is 96 to 100% full;|
|bit 3|(0x08): 该数据页是否存在鬼影记录|
|bit 4|(0x10): 是否是IAM页|
|bit 5|(0x20): 是否是混合页|
|bit 6|(0x40): 是否已分配使用|
|Bit 7|保留，未使用，无实际含义| 

## dbcc ind
> 语法
```
DBCC TRACEON(2588)
DBCC HELP('ind')
dbcc IND ( { 'dbname' | dbid }, { 'objname' | objid }, { nonclustered indid | 1 | 0 | -1 | -2 } [, partition_number] )
```
> 输出格式

- -2：返回所有IAM页，基于管理行内数据页，行溢出数据页及大对象数据页的IAM页
- -1：返回所有IAM页及数据页。
- 0：返回管理行内数据页的IAM页，行内数据页
- 1：返回聚集索引的数据页信息及IAM页信息（同-1）
- 2：返回第1个非聚集索引的数据页信息及IAM页信息
- 3：返回第2个非聚集索引的数据页信息及IAM页信息
- ...
- n：返回第（n-1）个非聚集索引的数据页信息及IAM页信息（n>1）

<center>![][1]</center>

## dbcc page

> 语法说明

```
DBCC TRACEON(2588)
DBCC HELP('PAGE')
dbcc PAGE ( {'dbname' | dbid}, filenum, pagenum [, printopt={0|1|2|3} ])
```

> 输出的格式有4种方式，不同方式，输出不一样。



- 0：输出可读形式的数据页页头数据
- 1：输出可读形式的数据页页头数据，并且还有槽位对应记录的十六进制内容
- 2：输出可读形式的数据页页头数据，输出整个数据页页头的十六进制数据，整一页的内容都显示，包括未使用的空间。
- 3：输出可读形式的数据页页头数据，并且包括记录中每个字段的可读形式，行溢出数据也会显示数据内容，但是大对象则不显示内容，而是说明其存储位置！所以选项3，也是输出内容最全面的。

## IAM
> Sytem_internals_allocation_units表存放第一个数据页和第一个IAM页的指针。IAM按照数据页的顺序存放数据页的指针。数据页之间并无直接链接。

<center>![][2]</center>

```
SELECT total_pages,used_pages,data_pages,
       first_page,root_page,first_iam_page
  FROM sys.system_internals_allocation_units
WHERE container_id ='72057594038779904'
```


  [1]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/608061-20170502183131101-852115051.png
  [2]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/882fba2bgab50aceb6a78&690.png