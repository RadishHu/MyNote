# Hive on Spark 调优

参考：[Hive on Spark性能调优](http://cwiki.apachecn.org/pages/viewpage.action?pageId=2888665)

## Yarn配置

`yarn.nodemanager.resource.cpu-vcores`和`yarn.nodemanager.resource.memory-mb`决定了Yarn可以使用的资源，这两值的设定取决于你节点的性能和非YARN应用跑在这些节点上的数量，一般工作节点只安装YARN NodeManager和HDFS DataNode 服务

## Spark配置

需要对Spark的executor和driver的内存、并行数量进行配置

### 配置Executor内存

