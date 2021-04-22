# MapReduce 基本概念及思想

**MapReduce程序的初衷是在数据存储节点运行，减少网络传输大量数据的现象.**

## MapReduce概述

MapReduce 是**一个分布式运算程序的编程框架**，MapReduce核心功能是将**用户编写的业务逻辑代码**和**自带默认组件**整合成一个完整的**分布式运算程序**，并发运行在一个Hadoop集群上。

#### 优缺点

**优点** 简单

* MapReduce易于编程

  **它简单的实现一些接口，就可以完成一个分布式程序**，这个分布式程序可以分不到大量廉价的PC机器上运行。

* 良好的扩展性

  当计算资源不能得到满足的时候，可以通过**简单的增加机器**来扩展计算能力。

* 高容错性

  MapReduce设计的初衷就是使程序能够部署在廉价的PC机器上，这要求它具有很高的容错性。比如**如果一台机器挂了，它就可以把上面的计算任务转移到另外一个节点上运行，不至于这个任务运行失败**，而且这个过程不需要人工参与，而完全有Hadoop内部完成。

* 适合PC级以上海量数据的离线处理

  可以实现上千台服务器集群并发工作，提供数据处理能力。

**缺点** 慢

* 不擅长实时计算

  MapReduce无法像MySQL一样，在毫秒级或者秒级内返回结果

* 不擅长流式计算

  流式计算的输入数据是动态的，而MapReduce的**输入数据是静态的**，不能动态变化。这是因为MapReduce自身的设计特点决定了数据源必须是静态的。

* 不擅长DAG（有向图）计算

  多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，MapReduce并不是不能做，而是使用后，**每个MapReduce作业的输出结果都会写入到磁盘，会造成大量的磁盘IO，导致性能非常低下**。

## MapReduce 应用开发

Configuration 类的实例代表配置属性及其取值的一个集合，后添加的文件的值会覆盖之前的值，当配置的属性 final 的值设为true时，其值不能被覆盖。

> 当覆盖一个 final 的值时会报一下警告：
>
> WARN [org.apache.hadoop.conf.Configuration] - configuration-2.xml:an attempt to override final parameter: xxx;  Ignoring.

### MapReduce进程

一个完整的MapReduce程序在分布式运行时有三个实例进程：

1. MRAppMaster：负责整个程序的过程调度及状态协调
2. MapTask：负责Map阶段的整个数据处理流程
3. ReduceTask：负责Reduce阶段的整个数据处理流程

### MapReduce编程规范

用户编写的程序分成三个部分：Mapper、Reducer和Driver。

**Manper阶段**：

1. 用户定义的Mapper要继承自己的父类
2. Mapper的输入数据是K-V对的形式，（KV的类型可自定义）
3. Mapper中的业务逻辑在map()方法中
4. Mapper的输出数据是K-V对的形式（KV的类型可定义）
5. map()方法（MapTask）对每一个<K,V>调用一次

**Reducer阶段**

1. 用户自定义的Reducer要继承自己的父类
2. Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
3. Reducer的业务逻辑写在reduce()方法中
4. ReduceTask进程对每一组相同k的<k,v>组调用一次reduc()方法

**Driver**

​	相当于YARN集群的客户端，用户提交整个程序到YARN集群，提交的是封装了MapReduce程序相关参数的job对象。

## MapReduce框架原理

### InputFormat数据输入

MapTask的并行度决定Map阶段的任务处理并发度，进而影响到整个Job的处理速度。

**MapTask并行度决定机制**：

* 一个Job的Map阶段并行度由客户端在提交Job时的切片数决定

* 每一个split切片分配一个MapTask并行实例处理
* 默认情况下，切片大小=BlockSize
* 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

> * **数据块**：Block是HDFS物理上把数据分成一块一块。
> * **数据切片**：数据切片只是在逻辑上对输入进行切片，并不会在磁盘上将其切分成片进行存储。