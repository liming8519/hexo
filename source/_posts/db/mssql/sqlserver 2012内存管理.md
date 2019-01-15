---
title: sqlserver 2012内存管理
tags:
  - sqlserver
categories:
  - db
date: 2019-01-15 01:00:00

---
> [原文地址：https://blogs.msdn.microsoft.com/apgcdsd/2013/07/10/sql-server-2012-memory-management](https://blogs.msdn.microsoft.com/apgcdsd/2013/07/10/sql-server-2012-memory-management/)
<!-- more -->

## 内存分配器的变化

> SQL Server 2012以前的版本，比如SQL Server 2008 R2等，有**single page allocator**和**multi page allocator**。 也就是说， 如果申请的内存是8k以内的，就会有单页分配器分配，而大于8kb的内存请求，使用multi page 分配器来管理。所以，如果你运行DBCC MemoryStatus，你会发现这两个分配器分配的内存情况。

<center>![dbcc memorystatus][1]</center>

> 如果你查询**memory clerk**，也会发现single pages 和multi pages 两列：

```
select * from sys.dm_os_memory_clerks
```

<center>![][2]</center>

> 而2012里面就不一样了，你会发现single page 和multi page字样消失了，只剩下pages 字样：

<center>![][3]</center>

> 而下面的语句的输出也是不一样的：

``` sql
select * from sys.dm_os_memory_clerks
```

<center>![][4]</center>

> 那么为什么会有这样的变化呢？原因就是SQL Server 2012里面不再有single page allocator 和multi page allocator，而是把它们统一起来了，叫做 any size page allocator。 下面的两张图可以看到这样的变化：

	SQL Server 2008 R2：

<center>![][5]</center>

	SQL Server 2012：

<center>![][6]</center>


 - [参考:https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2012/ms175019(v=sql.110)](https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2012/ms175019&#40;v=sql.110&#41;)

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  


  [1]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190115/6607.1.png
  [2]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190115/4812.2.png
  [3]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190115/5482.3.png
  [4]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190115/2577.4.png
  [5]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190115/5074.5.png
  [6]: https://raw.githubusercontent.com/zixujing/book1.github.io/master/image/20190115/7041.6.png