---
title: JVM监控和调优常用命令工具
tags:
  - java
categories:
  - java
date: 2019-04-01 02:00:00

---

> JVM监控和调优常用命令工具
> <!-- more -->

　　
## jps
命令格式：
jps [options] [hostid]
options参数解释：
- -l : 输出主类全名或jar路径
- -q : 只输出LVMID
- -m : 输出JVM启动时传递给main()的参数
- -v : 输出JVM启动时显示指定的JVM参数

最常用示例：
```
jps -l 输出jar包路径，类全名
jps -m 输出main参数
jps -v 输出JVM参数
```

## jinfo
命令格式：
jinfo [option] <pid>
options参数解释：
- -flag <name> 打印指定名称的参数
- -flag [+|-]<name> 打开或关闭参数
- -flag <name>=<value> 设置参数
- -flags 打印所有参数
- -sysprops 打印系统配置
- <no option> 打印上面两个选项


最常用示例：
```
#查看JVM参数和系统配置
jinfo 11666
jinfo -flags 11666
jinfo -sysprops 11666
#查看打印GC日志参数
jinfo -flag PrintGC 11666
jinfo -flag PrintGCDetails 11666
#打开GC日志参数
jinfo -flag +PrintGC 11666
jinfo -flag +PrintGCDetails 11666
关闭GC日志参数
jinfo -flag -PrintGC 11666
jinfo -flag -PrintGCDetails 11666
#还可以使用下面的命令查看那些参数可以使用jinfo命令来管理：
java -XX:+PrintFlagsFinal -version | grep manageable
```
常用JVM参数：
- -Xms：初始堆大小，默认为物理内存的1/64(<1GB)；默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制
- -Xmx：最大堆大小，默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制
- -Xmn：新生代的内存空间大小，注意：此处的大小是（eden+ 2 survivor space)。与jmap -heap中显示的New gen是不同的。整个堆大小=新生代大小 + 老生代大小 + 永久代大小。在保证堆大小不变的情况下，增大新生代后,将会减小老生代大小。此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8。
- -XX:SurvivorRatio：新生代中Eden区域与Survivor区域的容量比值，默认值为8。两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10。
- -Xss：每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K。应根据应用的线程所需内存大小进行适当调整。在相同物理内存下,减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。一般小的应用， 如果栈不是很深， 应该是128k够用的，大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:"-Xss is translated in a VM flag named ThreadStackSize”一般设置这个值就可以了。
- -XX:PermSize：设置永久代(perm gen)初始值。默认值为物理内存的1/64。
- -XX:MaxPermSize：设置持久代最大值。物理内存的1/4。


## jstat
命令格式：
jstat [option] LVMID [interval] [count]
其中LVMID是进程id，interval是打印间隔时间（毫秒），count是打印次数（默认一直打印）
option参数解释：
- -class class loader的行为统计
- -compiler HotSpt JIT编译器行为统计
- -gc 垃圾回收堆的行为统计
- -gccapacity 各个垃圾回收代容量(young,old,perm)和他们相应的空间统计
- -gcutil 垃圾回收统计概述
- -gccause 垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因
- -gcnew 新生代行为统计
- -gcnewcapacity 新生代与其相应的内存空间的统计
- -gcold 年老代和永生代行为统计
- -gcoldcapacity 年老代行为统计
- -gcpermcapacity 永生代行为统计
- -printcompilation HotSpot编译方法统计

```
常用示例及打印字段解释：
jstat -gcutil 11666 1000 3
```
字段解释:
S0 survivor0使用百分比
S1 survivor1使用百分比
E Eden区使用百分比
O 老年代使用百分比
M 元数据区使用百分比
CCS 压缩使用百分比
YGC 年轻代垃圾回收次数
YGCT 年轻代垃圾回收消耗时间
FGC 老年代垃圾回收次数
FGCT 老年代垃圾回收消耗时间
GCT 垃圾回收消耗总时间
```
jstat -gc 11666 1000 3
```
字段解释：
S0C survivor0大小
S1C survivor1大小
S0U survivor0已使用大小
S1U survivor1已使用大小
EC Eden区大小
EU Eden区已使用大小
OC 老年代大小
OU 老年代已使用大小
MC 方法区大小
MU 方法区已使用大小
CCSC 压缩类空间大小
CCSU 压缩类空间已使用大小
YGC 年轻代垃圾回收次数
YGCT 年轻代垃圾回收消耗时间
FGC 老年代垃圾回收次数
FGCT 老年代垃圾回收消耗时间
GCT 垃圾回收消耗总时间


## jstack　
命令格式：
jstack [-l] <pid> (连接运行中的进程)
jstack -F [-m] [-l] <pid> (连接挂起的进程)
jstack [-m] [-l] <executable> <core> (连接core文件)
jstack [-m] [-l] [server_id@]<remote server IP or hostname> (连接远程debug服务器)
option参数解释：
- -F 当使用jstack <pid>无响应时，强制输出线程堆栈。
- -m 同时输出java和本地堆栈(混合模式)
- -l 额外显示锁信息

```jstack -l 11666 | more```
输出信息：
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.25-b02 mixed mode):
"Attach Listener" #25525 daemon prio=9 os_prio=0 tid=0x00007fd374002000 nid=0x70e8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   Locked ownable synchronizers:
    None
    

## jmap
命令格式：
jmap [option] <pid> (连接正在执行的进程)
jmap [option] <executable <core> (连接一个core文件)
jmap [option] [server_id@]<remote server IP or hostname> (链接远程服务器)
option参数解释：
- <none> to print same info as Solaris pmap
- -heap 打印java heap摘要
- -histo[:live] 打印堆中的java对象统计信息
- -clstats 打印类加载器统计信息
- -finalizerinfo 打印在f-queue中等待执行finalizer方法的对象
- -dump:<dump-options> 生成java堆的dump文件
    　　　　dump-options:
    　　　　　　live 只转储存活的对象，如果没有指定则转储所有对象
    　　　　　　format=b 二进制格式
    　　　　　　file=<file> 转储文件到 <file>
- -F 强制选项
```
jmap -dump:live,format=b,file=dump.hprof 11666　
jmap -finalizerinfo 11666
jmap -heap 11666
jmap -histo:live 11666 | more
num     #instances         #bytes  class name
----------------------------------------------
1:         46608        1111232  java.lang.String
2:          6919         734516  java.lang.Class
3:          4787         536164  java.net.SocksSocketImpl
4:         15935         497100  java.util.concurrent.ConcurrentHashMap$Node
5:         28561         436016  java.lang.Object
```