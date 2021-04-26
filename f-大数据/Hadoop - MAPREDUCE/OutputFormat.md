MapReduce InputFormat

默认输出格式是 TextOutputFormat，把每条记录写为文本行。键和值可以是任意类型，因为 TextInputFormat 调用 toString() 方法把他们转化为字符串。没给键-值对由制表符进行分割。

也可通过 **mapreduce.output.textoutputformat.separator** 改变默认的分割符。

 