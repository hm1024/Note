# HDFS 运维

## HA

*hdfs haadmin*

```shell
Usage: haadmin
    [-transitionToActive <serviceId>]
    [-transitionToStandby <serviceId>]
    [-failover [--forcefence] [--forceactive] <serviceId> <serviceId>]
    [-getServiceState <serviceId>]
    [-checkHealth <serviceId>]
    [-help <command>]
```



启动 JournalNode 守护程序

`hadoop-daemon.sh start journalnode`

如果是从 非HA到HA则可以运行 `hdfs namenode -bootstrapStandby` 将确保JournalNode（由**dfs.namenode.shared.edits.dir**配置）包含足够的编辑事务以能够启动两个NameNode

将非HA NameNode转换为HA，则应运行命令“ *hdfs namenode -initializeSharedEdits* ”，该命令将使用本地NameNode edits目录中的edits数据初始化JournalNodes。

### ZKFC 相关

```
 bin/hdfs zkfc -formatZK
```

