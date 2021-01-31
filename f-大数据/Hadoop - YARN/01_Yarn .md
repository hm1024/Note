# Yarn  基本介绍

**Yarn 是什么？**

Yarn 分布式集群资源调度框架，调度内存和CPU，即调度算力资源。 

**为什么会有Yarn**？

> 在大数据发展的过程中，出现了多种计算框架，数据存储在HDFS中，
>
> 而在Hadoop1.0 中（没有Yarn 资源调度框架）服务器集群资源调度管理和 MapReduce 执行过程耦合在一起，如果想在当前集群中运行其他计算任务，比如 Spark 或者 Storm，就无法统一使用集群中的资源了。
>
> 在 Hadoop 早期的时候，大数据技术就只有 Hadoop 一家，这个缺点并不明显。但随着大数据技术的发展，各种新的计算框架不断出现，我们不可能为每一种计算框架部署一个服务器集群，而且就算能部署新集群，数据还是在原来集群的 HDFS 上。所以我们需要把 MapReduce 的资源管理和计算框架分开，这也是 Hadoop 2 最主要的变化，就是将 Yarn 从 MapReduce 中分离出来，成为一个独立的资源调度框架。

**Yarn 的架构图**

![af90905013e5869f598c163c09d718b1](01_Yarn .assets/af90905013e5869f598c163c09d718b1.jpg)

Yarn 包括两个部分：一个是资源管理器（Resource Manager），一个是节点管理器（Node Manager）。这也是 Yarn 的两种主要进程：**ResourceManager 进程负责整个集群的资源调度管理**，通常部署在独立的服务器上；**NodeManager 进程负责具体服务器上的资源和任务管理**，在集群的每一台计算服务器上都会启动，基本上跟 HDFS 的 DataNode 进程一起出现。

具体来说资源管理器有包括两个主要组件：

* **调度器**

  调度器其实就是一个资源分配算法，根据应用程序（Client）提交的资源申请和当前服务器集群的资源状况进行资源分配。Yarn 内置了几种资源调度算法，包括 Fair Scheduler、Capacity Scheduler 等，也可以开发自己的资源调度算法供 Yarn 调用。

  > 调度器其实就是一个资源分配算法，根据应用程序（Client）提交的资源申请和当前服务器集群的资源状况进行资源分配。Yarn 内置了几种资源调度算法，包括 Fair Scheduler、Capacity Scheduler 等，你也可以开发自己的资源调度算法供 Yarn 调用。

* **应用程序管理器**

  应用程序管理器负责应用程序的提交、监控应用程序运行状态等.应用程序启动后需要在集群中运行一个 ApplicationMaster，ApplicationMaster 也需要运行在容器里面。每个应用程序启动后都会先启动自己的 ApplicationMaster，由 ApplicationMaster 根据应用程序的资源需求进一步向 ResourceManager 进程申请容器资源，得到容器以后就会分发自己的应用程序代码到容器上启动，进而开始分布式计算。



