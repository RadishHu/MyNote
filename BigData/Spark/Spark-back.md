# Spark

# 目录

> - [Spark 简介](#chapter1)
> - [Spark 安装](#chapter2)

# Spark 简介 <a id="chapter1"></a>

Spark 是一个计算框架，它提供了基于  Java、Scala、Python 和 R  的高级 API，并且支持通过 SQL 进行计算的 Spark SQL、用于机器学习的 MLlib、用于图计算的 GraphX 和进行流计算的 Spark Streaming

Driver Program，每个 Spark 程序都包含一个 driver 程序，用来运行用户定义的 `main` 函数，并执行各种并行的操作。

RDD，Resilient Distributed Dataset，是一个带有分区的数据集合，分布在集群各个节点上，可以被并行操作。RDD 可以通过文件创建，也可以通过 driver 程序中 Scala 集合创建 ，还可以通过其他的 RDD 转换而来。用户可以把 RDD 持久化到内存中，来重复使用。如果节点挂掉，RDD 可以自动恢复回来。

共享变量，共享变量是 Spark 的另一个数据抽象，它可以在并行计算中使用。默认情况下，当 Spark 运行一个函数时，是以 task 集合的形式并行运行在不同节点上，它把函数中使用到的变量拷贝到每个 task 中。有时一个变量需要在各个 task 之间或者 task 和 driver 程序之间共享。Spark 支持两种共享变量：

- broadcast variables，广播变量
- accumulators，累加器



# Spark 安装 <a id="chapter2"></a>

在安装 Spark 之前需要安装 Java 8+, Scala 2.1+

下载 Spark 

<https://spark.apache.org/downloads.html>

## 编译 Spark

参考：[《Building Spark》](<http://spark.apache.org/docs/latest/building-spark.html>)

## Windows 安装 Spark

Spark 可以运行在 Windows 系统

# Spark Shell

通过 Spark Shell 可以交互的执行 Spark  程序，这样可以更加有效的学习 Spark API

```
./bin/spark-shell --master local[2]
```

> --master 指定集群的主节点的 URL，或通过 local[N] 在本地开启 N 个线程来运行



# Spark on Yarn

Spark 支持多种资源调度：

- Standalone Deploy Mode
- Hadoop Yarn
- Apache Mesos
- Kubernetes

