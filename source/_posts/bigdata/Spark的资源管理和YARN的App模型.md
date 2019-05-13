---
title:  Spark的资源管理和YARN的App模型
tags:
- nginx 
categories: 
- linux 
date: 2019-05-13 02:00:00
---
> Spark的资源管理和YARN的App模型
<!-- more -->

## 一个关于在YARN下运行 Spark 和 MapReduce 如何管理资源的简单介绍

在 YARN 上出现最多的应用除了 MapReduce 之外就是 SPARK，在 Cloudera，我们花了很大的精力在稳定 Spark-on-YARN，在 CDH 5.0.0 上添加了 Spark on YARN 集群的支持。
在这篇文章中，你将了解到 Spark 和 MapReduce 结构的不同，为什么你需要关心这些已经他们如何在 YARN 的 ResourceManager 上运行的。
## 应用
在 MapReduce 中，最高级的计算单元是 Job。系统载入数据，对数据使用 map function，再对数据进行 shuffle，在对数据使用 reduce function，最后将数据写回磁盘。Spark 有相似的 Job 的概念（只是在每个 Job 中相比 map 和 reduce 包含更多的 stage），但是它有一个更加高级的概念，叫做 应用（application)，应用可以以并行或者串行的方式同时运行多个 Job。
熟悉 Spark API的用户知道，一个 应用 对于一个 SparkContext 的实例。一个 应用 可以用于单个 Job，或者分开的多个 Job 的 session，或者响应请求的长时间生存的服务器。与 MapReduce 不同的是，一个 应用 的进程（我们称之为 Executor)，会一直在集群上运行，即时当时没有 Job 在上面运行。这样实现的目的在于支持在内存中保持数据用于快速访问和闪电般的 task 启动速度。

## Executors
MapReduce 的每个 task 对应一个进程，当 task 结束之后，进程也随之结束。在 Spark 中很多个 task 可以在一个进程中并行地执行，而且这个进程得声明周期与整个 Spark 应用一致，即使没有任何 Job 在运行。
这种模型的优势在上面提到过，就是速度。每个 task 可以非常快地启动、访问内存中的数据。劣势在于粗粒度的资源管理。当一个 应用 启动时，executor 的个数已经确定了，每个 executor 分配了固定的资源，在这个 应用 的声明周期里面就一直占用着这些资源。（一旦 YARN 支持 container resizing，我们就会在 Spark 上应用它使得支持资源动态分配）。
## Active Driver
Spark 使用一个有效的 driver 进程管理 Job 的流以及调度 task。一般情况下，这个 driver 进程跟客户端进程一样用于初始化 Job，但是在 YARN 模式中，driver 进程可以在集群上运行（后面会详细介绍）。与此不同的是，即使客户端经常停止了之后 MapReduce 的 Job 可以继续执行。在 Hadoop 1.x 中， JobTracker 负责 task 的调度，在 Hadoop 2.x 中则由 application master 负责调度。

## 可替换的资源管理
Spark 支持可替换的集群管理，集群的管理器负责启动 executor 进程。 Spark 程序的开发人员则不需要关心自己的程序使用的是什么集群管理器。
Spark 支持 YARN、Mesos 以及它自带的 standalone 资源管理器。所有这三种框架都包含两个组件。一个中心的管理服务（YARN 的 ResourceManager，Mesos 的 master 以及 Spark 的 standlone master）决定哪些 应用 可以启动 executor 进程，以及在何时何处启动。一个在每个节点上运行的从服务（YARN 的 NodeManager，Mesos 的 slave 以及 Spark 的 standlone slave），它才是真正的启动 executor 进程。它也同时监控进程是否存活已经资源消耗。
## 为什么要在 YARN 上运行
相比于在 Mesos 和 Standalone，在 YARN 运行有以下几个优势：
在YARN可以支持同时管理在它上面运行的不同的框架的资源，你可以先把所有的资源运行一个 MapReduce Job，然后再把一部分资源给 Impala 做查询同时另一部分给 Spark 应用，实现这些不需要更改任何配置。
你可以利用 YARN 调度去的所有优点用户对 应用 进行分类、隔离以及分优先级。
只有 YARN 为 Spark 提供了安全性。在 YARN 上，Spark 可以在 Kerberized Hadoop 集群上从而可以再进程间使用安全授权。
## 在 YARN 上运行
当 Spark 在 YARN 上运行时，每个 executor 以 YARN 容器的方式运行。MapReduce 会为每一个 task 调度一个容器再起一个 JVM，Spark 在同一个容器中维护多个 task。这样的实现方式可以使得 task 的启动速度有量级上的提升。
Spark 在 YARN 上运行时支持两种方式，“yarn-cluster”模式和”yarn-client“模式。一般情况下，yarn-cluster 模式更适合线上环境，yarn-client 模式更适合用户交互和调试环境。
需要了解这两者之间的不同需要先理解 YARN Application master 的概念。在 YARN 中，每个 应用 都有一个 Application master 进程，这个启动这个应用的第一个容器。这个进程负责向 ResourceManager 请求资源，当资源分配完成之后，向 NodeManager 发送启动容器的指令。Application master* 避免使用需要一直运行的client。即使启动这个应用的进程退出了，任务还是会有在集群上运行的 YARN 继续执行。
在 yarn-cluster 模式中， driver 运行 Application master，这就意味着这个进程需要同时进程驱动应用以及从 YARN 请求资源，并且这个进程在一个 YARN 容器中运行。启动这个应用的进程并不需要在整个应用的运行周期内一直运行着。 
然而在 yarn-cluster 模式并不适合需要交互地使用 Spark。有些 Spark 应用 需要用户的输入，比如 spark-shell 和 pyspark，需要 Spark driver 在用于初始化 Spark 应用的 client 进程中执行。在 yarn-client 模式中， Application master 仅仅是用于向 YARN 请求资源容器，client 进程直接与这些容器进行通信以进行调度。