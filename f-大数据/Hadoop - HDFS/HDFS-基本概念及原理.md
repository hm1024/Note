# HDFS-基本概念及原理

## NameNode 的实现

NameNode 节点维护着 HDFS 文件系统中两个最重要的关系：

* HDFS 文件系统的文件目录树 ，以及文件的数据块索引，即每个文件对应的数据块列表。
* 数据块和数据节点的对应关系，即某一数据块 与 数据节点的对应关系。

HDFS 的目录树、 元信息和数据块索引等信息会持久化到磁盘上，保存在 FSImage 和 EditLog中。数据块 和 数据节点的对应关系则在NameNode 启动后，由 DataNode 上报，动态建立。































参考资料：

* 《Hadoop 技术内幕：深入解析 Hadoop Common 和 HDFS 架构设计与实现原理》