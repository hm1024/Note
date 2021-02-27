# Java NIO



## 缓冲区

缓冲区的四个核心属性：

> mark <= position <= limit <= capacity

* mark：标记，表示记录当前 position 的位置。可以通过 reset() 恢复到 mark的状态
* position：位置，表示缓冲区中正在操作数据的位置

* limit：界限，表示缓冲区可以操作数据的大小。（limit 后数据不能进行读写）
* catpacity：容量，表示缓冲区最大存储数据的容量，一旦声明不能改变。



