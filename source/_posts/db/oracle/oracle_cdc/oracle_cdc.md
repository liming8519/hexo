---
title: oracle cdc
tags:
  - oracle
categories:
  - db
date: 2019-01-08 00:00:00
---
# 摘要
> oracle也具有cdc的功能，原理如下。
<!-- more -->
# Oracle CDC 简介
   很多人都认为，只要是涉及到数据库数据复制和增量数据抽取，都是需要购买收费软件的。实际上，我们通过Oracle提供的CDC和LogMiner等免费工具也能实现数据库数据复制和增量数据抽取，各种数据复制软件只是使得获取增量数据更加便捷，或者是可以支持更多的扩展功能（例如：异构数据库之间的同步，ETL过程的数据清洗、装换），但实际Oracle本身是支持CDC机制，只是很少有人关注，操作起来也有些复杂，而且据传言并不稳定，常常见到论坛上爆出一些莫名其妙的问题。
## Synchronous Change Data Capture Configuration（同步复制）
   ![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/1068207-20170526192306372-1819930635.png)

   原理很简单，原表、目标表必须是同一个库，采用触发器的机制（设置同步CDC后，并看不到触发器，但实际运行机理还是触发器的机制）将原表内容复制到另一个目标表。这个机制就不多说了，和自己给表建触发器没什么太大差别。
## Asynchronous HotLog Configuration（异步在线日志CDC）
  ![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/1068207-20170526192327279-376093983.png)

   这个过程已经没有触发器了，而是使用Redo Log，但是使用在线日志，并不是归档日志。并且原表、目标表仍然必须是同一个库。这种模式是相对简单的，同时这种模式是在Oracle 10以上才产生的，9i是没有这个机制的。
## Asynchronous Distributed HotLog Configuration（异步分布式CDC）
  ![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/1068207-20170526192342669-1522054385.png)
  实际这个模式是对异步在线日志CDC的一种优化，也比较容易理解，就是加入了DB-LINK机制，使原表、目标表不在同一个数据库。实际是和异步在线日志CDC没有什么本质区别。
## Asynchronous Autolog Online Change Data Capture Configuration（异步在线日志复制CDC）
  ![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/1068207-20170526192358513-964979174.png)
  异步在线日志复制CDC模式就要高级很多了，使用Standby Redo Log（热备数据库日志），实际就是使用Oracle的热备机制，将日志写入了热备数据库，目标表就可以建立在热备库上，这对主数据库性能影响就进一步降低。
## Asynchronous AutoLog Archive Change Data Capture Configuration（归档日志CDC）
  ![](https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/1068207-20170526192417247-198841881.png)
  归档日志CDC模式是最完美的模式，但是需要有机制可以获取归档日志（并行文件系统技术），然后在目标端分析归档日志进行变化数据处理，这种模式理论上来讲，几乎可以完全不影响原数据库的性能。
  坦白来说，我对Oracle理解并不深，只是为了解决特定的几个问题多看了一点，在现实工作中遇到类似问题需要解决的，或对技术痴狂的同学可以研究一下，我贴上了4种模式具体的设置步骤，虽然是英文的，但是还是非常明确的。（我比较推荐使用第二种，因为设置比较简单，性能上也属于中规中矩，如果没有什么特别要求，可以采用异步在线日志CDC。
