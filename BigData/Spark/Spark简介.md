# Spark简介

## 目录

> - [Spark 运行模式](#chapter1)

Spark生态系统包含多个子项目：Spark SQL、SparkStreaming、GraphX、MLlib



## Spark 运行模式 <a id="chapter1"></a>

- Local



## Spark架构

Spark主要由三部分组成：Spark Driver、Cluster Master(YARN、Standalone或Mesos)、Cluster Worker(Executor)

- Spark Driver

  驱动程序(Driver Program)包含应用的main函数，并且定义了集群上的分布式数据集，还有对分布式数据集的相关操作

  driver program通过SparkContext对象访问Spark

  driver program负责把用户程序转化为多个物理执行单元

  deiver program在各个执行器进程间协调任务的调度

- Executor负责在Spark作业中运行任务

  负责组成Spark应用的任务，并将结果返回给驱动器进程

  通过自身的块管理器(Block Manager)为用户程序中要求缓存的RDD提供内存式存储

## RDD

- RDD，Resilient Distributed Dataset，弹性分布式数据集

  RDD是Spark对数据的核心抽象，它代表一个不可变、可分区、里面元素可并行计算的集合

  - Dataset：一个数据集合，用于存放数据
  - Distributed：RDD的数据时分布式存储的，可用于分布式计算
  - Resilient：RDD的数据可以存储再内卒中或磁盘中

- RDD属性

  - A list partitions：一个分区列表

    RDD每个分区都会被一个计算任务处理，可以在创建RDD时指定RDD的分区个数，如果没有指定，就采用默认，读取HDFS上的文件产生的RDD分区数跟block个数相等

  - A function for computing each split：一个计算每个分区的函数

    Spark中RDD的计算是以分区为单位的

  - A list of dependencies on other RDDS：一个RDD会依赖于其它多个RDD，RDD之间的依赖关系

    RDD的每次转换都会生成一个新的RDD，在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据

  - A partitioner for key-value RDDS：一个Partitioner，即分区函数(可选)

    Spark中实现了两种类型的分区函数：基于哈希的Hash Partitioner、基于范围的RangeParitioner。只有key-value型的RDD才有Partitioner，非key-value的RDD的Parittioner的值为None。Partitioner函数决定了parent RDD Shuffle输出时的分区数量

  - A list of preferred locations to compute each split on：一个列表，存储每个partiton的优先位置(可选)

    这个列表保存的是每个Partition所在的块的位置，Spark在进行任务调度的时候，会尽可能地将计算任务分配到其所要处理数据块存储的位置

- 创建RDD的方法

  - 读取一个外部数据集
  - 使用驱动程序的集合对象

- RDD算子

  - Transformation，转换操作

    由一个RDD生成一个新的RDD，转换操作时惰性的，只有在第一次Action被调用时，才会真正计算

  - Action，行动操作

    会对RDD计算出一个结果，并把结果返回到驱动程序中，或把结果存储到外部存储系统中喔咕

- RDD依赖关系

  - 窄依赖，每一个父RDD的Partition最多被子RDD的一个Partition使用
  - 宽依赖，多个子RDD的Partition会依赖同一个父RDD的Partition

  Lineage(血统)，记录RDD的元数据信息和转换操作，当RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区

- RDD持久化(缓存)

  Spark RDD是惰性求值的，对RDD调用Action，Spark每次都会重算RDD以及它的所有依赖，为了避免多次计算同一个RDD，可以对数据进行持久化。

  RDD可以通过persist()或cache()将前面的计算结果缓存，但并不是这两个方法被调用时立即缓存，而是触发action时。

  cache()最终调用了persist()，默认存储级别是仅在内存存储一份

  持久化级别：

  - MEMORY_ONLY
  - MEMORY_AND_DISK，如果数据在内存放不下，溢写到磁盘
  - DISK_ONLY

  > 末尾添加：
  >
  > * _SER，对RDD进行序列化后，再持久化
  > * _2，把持久化数据存两份

## checkpoint

对于重复使用的数据，采用persist把数据放在内存，虽然快，但是不可靠，checkpoint将数据放在HDFS上，完成最大化的可靠的持久化数据

## DAG

DAG，Directed Acyclic Graph，又向无环图，RDD进行的一系列操作称为DAG

## Spark任务调度

