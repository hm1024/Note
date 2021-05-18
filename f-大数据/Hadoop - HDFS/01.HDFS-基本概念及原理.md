# HDFS-基本概念及原理

## NameNode 的实现

NameNode 节点维护着 HDFS 文件系统中两个最重要的关系：

* HDFS 文件系统的文件目录树 ，以及文件的数据块索引，即每个文件对应的数据块列表。
* 数据块和数据节点的对应关系，即某一数据块 与 数据节点的对应关系。

HDFS 的目录树、 元信息和数据块索引等信息会持久化到磁盘上，保存在 FSImage 和 EditLog中。数据块 和 数据节点的对应关系则在NameNode 启动后，由 DataNode 上报，动态建立。

## INode



### INodeFile

NameNode，文件由 INodeFile 抽象，是 INode 的子类。INodeFIle 包含两个文件特有的属性

* header：在一个长整型变量里保存文件的副本系数和文件数据块的大小，它的高16字节存放着副本系数，低48位存放了数据块大小。

* blocks：保存文件拥有的数据块，数据元素类型是 BlockInfo，

  BlockInfo 包含

  * 数据块所属文件，即文件的 INodeFile 对象；
  * Block副本的 DataNode节点，
  * 数据块和文件、数据块和数据节点两个对应关系































参考资料：

* 《Hadoop 技术内幕：深入解析 Hadoop Common 和 HDFS 架构设计与实现原理》