# MapReduce InputFormat

## MapReduce中的InputFormat类型

### **InputFormat**

从InputFormat类图看，InputFormat抽象类仅有两个抽象方法：

* List<InputSplit> getSplits()：根据输入文件计算出输入切片(InputSplit)，解决输入文件切片问题
* RecordReader<K,V> createRecordReader()：创建 ResordReader,从 InputSplit 中读取数据，解决从切片中读取数据的问题

在 调用 getSplits() 获取切片时，还会验证输入文件是否可分割、文件存储时分块的大小和文件大小等因素，通过 InputFormat，MapReduce 框架可以做到：

* 验证作业输入的正确性
* 将输入文件切割成**逻辑分片**(InputSplit)，一个InputSplit将会被分配给一个独立的MapTask
* 提供RecordReader实现，读取InputSplit中的`K-V对`供Mapper使用

<img src="images/InputFormat/7126254-348de605cbcf0bd8.png" alt="7126254-348de605cbcf0bd8"  />

### FileInputFormat

FileInputFormat 是所有基于文件的InputFormat的基类，指定数据文件所在的输入目录（或文件，输入既可以指定为目录，也可以指定为文件）。 FileInputFormat将读取所有文件并将这些文件分成一个或多个InputSplits。

###  TextInputFormat:dart:

TextInputFormat 是MapReduce的默认InputFormat。TextInputFormat将每个输入文件的每一行视为单独的记录，并不执行分析。对于未格式化的数据或基于行的记录（如日志文件）非常有用。

* Key  - 是文件内行开头的字节偏移量（不是整个文件只是一个inputsplit），因此如果与文件名一起使用，它将是唯一的。
* Value – 是该行的内容，不包括行结束符

### NLineInputFormat

NLineInputFormat是TextInputFormat的另一种形式，将 N 行作业一个切分 split，默认情况下 N = 1，即输入文件中的每行起一个Mapper来处理，

* Key - 行的字节偏移量
* Value - 行的内容。

### KeyValueTextInputFormat

KeyValueTextInputFormat 与TextInputFormat类似，它也将每行输入视为单独的记录，但KeyValueTextInputFormat通过制表符`/t`将行本身分解为键和值。

* Key -  行首直到制表符
* 制表符后的行剩余部分

### SequenceFileInputFormat

SequenceFileInputFormat是一个读取序列文件的InputFormat。序列文件是存储二进制键值对序列的二进制文件，序列文件是块压缩的，并提供几种任意数据类型（不仅仅是文本）的直接序列化和反序列化

键和值都是用户自定义的

### SequenceFileAsTextInputFormat

SequenceFileAsTextInputFormat 是 SequenceFileInputFormat 的另一种形式，它将序列文件键值转换为Text对象，通过调用`tostring()`转换是在键和值上执行的，这个InputFormat使序列文件适合输入流。

### DBInputFormat

 DBInputFormat 使用JDBC从关系数据库中读取数据。







**参考链接**：

* [Hadoop InputFormat介绍](https://www.jianshu.com/p/12c66b6f5c57)