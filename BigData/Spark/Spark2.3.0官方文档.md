# Spark

Apache Spark 是一个分布式计算系统，它支持多种 API：Java, Scala, Python 和 R，并且包含多种工具：通过 sql 处理结构化数据的 **Spark SQL**, 用于机器学习的 MLlib, 用于图计算的 GraphX 和 进行进行流计算的 Spark Streaming。

Spark 运行在 Java8+, Python 2.7+/3.4+, R 3.1+，对于 Scala API，Spark 2.2.0 使用的是 Scala 2.11，我们可以使用 2.11.X。

在 Spark 2.2.0 版本中，移除了对 java 7，python 2.6 和 Hadoop 小于 2.6.5 版本的支持。

## 基础概念

- Driver Program, 每个 Spark 程序都包含一个 driver 程序，用来运行用户定义的 `main` 函数，并执行各种并行的操作。

- RDD, Resilient Distributed Dataset, 是一个带有分区的数据集合，分布在集群各个节点上，可以被并行操作。RDD 可以通过文件创建，也可以通过 driver 程序中 Scala 集合创建，还可以通过其它 RDD 转换而来。用户可以把 RDD 持久化到内存中，来重复使用。如果节点挂掉，RDD 可以自己恢复回来。

  > 在 Spark 2.0 之前，Spark 主要编程接口是 RDD。在 Spark 2.0 之后，RDD 被 Dataset 代替，它像 RDD 一样是强类型，但是在计算引擎上有更丰富的优化。RDD 接口仍然支持，

## Spark Shell

Spark Shell 是学习 API 的一个便捷方式，并且是一个进行交互的分析数据的好方法，Spark Shell 中可以使用 Scala 或 Python API。可以通过下面的命令开启 Spark Shell:

```shell
# scala
./bin/spark-shell

# python
./bin/pyspark

# spark on yarn 
./bin/spark-shell --master yarn --deploy-mode client
```

## Caching

Spark 也支持把数据集缓存到集群的内存中，当数据被多次访问时，这个方法是非常有用的。

```shell
linesWithSpark.cache()
```

# RDD 编程

## 概述

每个 Spark 应用程序都包含一个 driver 程序，它用于运行用户的 main 函数和在集群上执行各种并行操作。Spark 的主要数据抽象是 RDD，它是一个分布在集群各节点的数据集合，可以被并行的操作。RDD 可以通过文件创建，也可以通过 driver 程序中的 Scala 集合创建，还可以通过其它 RDD 转化而来。用户可以把 RDD 持久化在内存中，这样就可以被重用。RDD 也可以在节点挂掉后恢复过来。

## 连接 Spark

Spark 2.3.0 是在 Scala 2.11 基础上构建的，通过 Scala 写 Spark 应用程序，需要使用 Scala 的兼容版本(比如 2.11.X)。

写 Spark 程序，需要在 Maven 中引入一下依赖：

```properties
groupId = org.apache.spark
artifactId = spark-core_2.11
version = 2.3.0
```

如果要使用 HDFS 集群，需要配置 *hadoop-client* 依赖：

```properties
groupId = org.apache.hadoop
artifactId = hadoop-client
version = <your-hdfs-version>
```

需要在程序中引入一下 Spark 类：

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf
```

> 在 Spark 1.3.0 版本之前，需要使用 import org.apache.spark.SparkContext._ 

## 初始化 Spark

写 Spark 程序的第一件事是创建 `SparkContext` 对象，这个对象告诉 Spark 如何访问集群，在创建 `SparkContext` 前需要先创建一个 `SparkConf` 对象，它包含了应用程序的配置。

每个 JVM 只能有一个 `SparkContext` 存活，因此在创建一个新的 `SparkContext` 前必须使用 `stop()` 来终止 `SparkContext`。

```scala
val conf = new SparkConf().setAppName(appName).setMaster(master)
new SparkContext(conf)
```

> appName，是在集群 UI 界面显示的应用程序的名字
>
> master，Spark、Mesos、YARN 集群 URL 或 local。在实际工作中，一般不会在代码中指定 *master*, 而是在使用 *spark-submit* 命令式指定，但是在测试和开发时可以在程序中指定 *master* 为 "local"

### 使用 Shell

在 Spark Shell 中 SparkContext 已经被创建好了，可以通过变量 `sc` 来引用，自己创建的 SparkContext 是不管用 的。

示例：

指定 4 个核来运行 spark-shell

```shell
$ ./bin/spark-shell --master local[4]
```

添加 code.jar 到类路径下

```shell
$ ./bin/spark-shell --master local[4] --jars code.jar
```

使用 maven 坐标来添加依赖

```shell
$ ./bin/spark-shell --master local[4] --packages "org.example:example:0.1"
```

完整的选项说明，可以通过 *spark-shell --help* 获取

## Resilient Distributed Datasets (RDDs)

Spark 编程都是围绕着 RDD 这个概念来进行的，它是一个具有容错性的可以被并行操作的数据集合。有两种方式来创建 RDD：

- 并行化 driver 程序中已经存在的集合
- 或从外部的存储系统中创建

### 并行化集合

可以通过 SparkContext 的 `parallelize` 方法来并行化 driver 程序中的集合，这个集合会被复制成一个可以进行并行操作的分布式数据集。示例如下：

```scala
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
```

有一个重要的参数是设置分区数，Spark 会给每个分区分配一个 task。通常情况下，也可以给每个 CPU 上分配 2-4 个分区。一般 Spark 会基于集群自动设置分区数，当然也可以通过 `parallelize` 手动设置(比如，sc.parallelize(data,10))。一些地方会使用切片(slices) 来表示分区(partition)。

### 外部的数据集合

Spark 可以从任何支持 Hadoop 的外部存储系统中创建 RDD (比如，本地文件系统、HDFS、HBase等).

通过文本文件创建 RDD，可以使用 SparkContext 的 `textFile` 方法，这个方法需要提供文件的 URI，并把文件按行拆分成集合。示例：

```scala
scala> val distFile = sc.textFile("data.txt")
distFile: org.apache.spark.rdd.RDD[String] = data.txt MapPartitionsRDD[10] at textFile at <console>:26
```

> - 如果使用的是本地文件系统，那 worker 节点的相同路径上访问该文件。同时也要可以复制文件到所有的工作节点或使用网络挂载共享文件系统。
> - Spark 的所有读取文件方法，包括 *textFile* 方法，也支持读取目录、压缩文件和通配符。比如，我们可以使用 textFile("/my/directory"), textFile("/my/directory/*.txt"), textFile("/my/directory/\*.gz")
> - *textFile* 方法可以在第二个选项添加一个参数来控制分区数。默认情况下，Spark 会为文件的每个 block 创建一个分区，我们可以设置比 block 数更大的值，但是不可以设置比 block 数更小的值。

除了文本文件，Spark 的 Scala API 也支持一下数据格式：

- *SparkContext.wholeTextFiles* 可以从包含多个小文件的目录中读取数，并把每个文件返回成一个 filename-content 对。这是跟 textFile 不同所在，textFile 会把文件的每一行作为一条记录。分区数由数据所在的位置决定，在一些情况下，会导致分区数过少。为了解决这种情况，*wholeTextFiles* 提供一个第二参数来控制分区的最小值。
- *SparkContext.sequenceFile[K, V]* 方法可以读取序列化文件，k 和 v 分别是文件的 key 和 values。它们应该是 Hadoop Writable 接口的子类，比如 IntWritable 和 Text。Spark 允许你为一些常见的 Writable 指定本地类型，比如，*sequenceFile[Int, String] 会自动读取 IntWritables 和 Text。
- 对于其它的 Hadoop 数据格式，可以使用 *SparkContext.hadoopRDD* 方法，这个方法需要传入 *jobConf* 和 input format 类、key 类和  value 类。也可以使用 *SparkContext.newAPIHadoopRDD* 来读取基于 MapReduce API (org.apache.hadoop.mapreduce) 的输入格式。
- *RDD.saveObjectFile* 和 *SparkContext.objectFile* 支持以 Java 序列化对象的简单格式保存 RDD。这个没有 Avro 这样的序列化格式高效，这种格式提供了一个简单的方法来保存任何 RDD。

### 操作 RDD

RDD 支持两种操作：

- transformations, 从一个已经存在的数据集创建一个新的数据集。比如，*map* 是一个 transformation 操作，它对数据集中的每个元素进行操作生成一个新的 RDD。
- actions, 对数据集进行计算后把结果返回到 driver 程序。比如，*reduce* 是一个 action 操作，它聚集数据集中的所有元素，并使用函数生成最终的结果到 driver 程序。

所有的 transformations 操作都是懒惰的，它们不会立马去执行操作，只是记录作用在一些基础数据集上的 transformations 操作，只有当 action 操作需要返回结果到 driver 程序时才会执行 transformations 操作，这个机制保障了 Spark 更加高效的运行。

默认情况下，当每次运行 action 操作时，transformation 操作的 RDD 都会被重新运行一次。你可以通过 *persist* 或 *cache* 方法持久化 RDD 到内存，这样下次可以更快的访问。

#### Basic

使用下面的示例，来阐明 RDD 基础：

```scala
val lines = sc.textFile("data.txt")
val lineLengths = lines.map(s => s.length)
val totalLength = lineLengths.reduce((a, b) => a + b)
```

第一行从外部文件定义了一个基础 RDD，这个数据集不会加载到内存中，*lines* 只是一个指向文件的指针。第二行定义的 *lineLengths* 是 *map* transformation 操作的结果，*lineLengths* 不会立马被计算。最后运行 *reduce* action 操作，这时 Spark 才会把计算拆分为 task ,运行在不同的机器上，每台机器都只会运行它们自己本分的 *map* 和本地 reduction，返回它们自己的结果到 driver 程序。

如果想在之后再次使用 *lineLengths*，我们可以添加：

```scala
lineLengths.persist()
```

> 在 *reduce* 之前，会把 *lineLengths* 保存到内存中

#### 传递函数给 Spark 

Spark API 非常依赖把 driver 程序中的函数传递到正在运行的集群上，有两种推荐方法来实现这个：

- 匿名函数表达式，这个可以通过非常简短的代码来实现

- 在全局唯一的对象中定义方法，比如，你可以定义 *object MyFunctions* , 然后传递 *MyFunction.func1* ，如下：

  ```scala
  object MyFunctions {
      def func1(s: String): String = {...}
  }
  
  myRdd.map(MyFunction.func1)
  ```

  虽然可以将一个引用传递给类实例中的方法(而不是单例对象)，但这需要传递包含该对象的类和方法，比如：

  ```scala
  class MyClass {
      def func1(s: String): String = { ... }
      def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(func1)}
  }
  ```

  如果我们在这里创建一个 *MyClass* 实例，并调用 *doStuff* 方法，那么 *map* 会调用 *MyClass* 实例的 *func1* 方法，因此需要将整个对象发送到集群，这个跟 *rdd.map(x => this.func1(x))* 类似。

  同样，访问外部对象中的字段，会引用整个对象：

  ```scala
  class MyClass {
      val field = "Hello"
      def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(x => field + x) }
  }
  ```

  这个类似于写 *rdd.map(x => this.field + x)*，会引用这个 *this*。为了避免这种问题，最简单的方法是复制 *field* 到本地变量，而不是在外部访问：

  ```scala
  def doStuff(rdd: RDD[String]): RDD[String] = {
      val field_ = this.field
      rdd.map(x => field_ + x)
  }
  ```

#### 理解闭包

Spark 的一个难点就是理解在集群中执行代码时变量和方法的作用范围和生命周期。RDD 的操作修改其作用范围外的变量，经常会引起混淆。下面的示例会使用 *foreach()* 来递增一个计数器，相似的问题也会发生在其他操作中。

考虑下面 RDD 元素的总和，根据计算是否在同一 JVM 中进行，它的执行结果可能会不同。一个常见的示例是通过 *local* 模式(--master = local[n])运行 Spark 跟通过集群模式(通过 spark-submit 提交到 YARN)运行 Spark 进行对比：

```scala
var counter = 0
var rdd = sc.parallelize(data)

// 不要做这样的操作
rdd.foreach(x => counter += x)

println("Counter value: " + counter)
```

**本地模式跟集群模式对比：**

上述代码可能不会按预期运行，为了执行 job，Spark 会将 RDD 操作到拆分为 task，每个 task 都会通过一个 executor 运行。在执行前，Spark 会计算任务的闭包，闭包是那些变量和方法，它们必须是可访问的，以便 executor 执行 RDD 的计算(在这个示例中是 *foreach()*)，这个闭包是序列化的并发送到每个 executor。

发送给每个 executor 的闭包变量是副本，因此，当 ***counter*** 被 *foreach* 函数引用，它不再是 driver 节点的 ***counter***。在 driver 节点的内存中仍然存在一个 ***counter***，但是 executor 不能再访问它，executor 只能看到序列化闭包的副本。最后 ***counter*** 的值仍然是 0, 因为所有对 ***counter*** 的操作都引用了序列化闭包内的值。

在本地模式下，*foreach* 函数在一些情况下会跟 driver 在同一个 JVM 中运行，并且会引用同一个 ***counter***，并可以更新它的值。

在这些场景中，可以使用累加器(Accumulator)，累加器提供了一种机制，当程序在集群中的多个节点上运行时，确保变量安全更新。

通常情况下，闭包 - 类似于循环或本地定义的方法，不应该用来改变全局状态。Spark 无法保证从闭包外引用的对象的修改行为。执行此操作的一些代码可以在 local 模式下运行，但是这只是偶然情况，并且这类代码在集群模式下不会按照预期运行。如果需要全局聚合，可以使用累加器。这个操作可能会导致 driver 的内存不足，因为 *collect()* 会把整个 RDD 收集到一台机器上。如果你只是想打印 RDD 中的部分元素，可以使用 *take()* 方法：*rdd.take(100).foreach(println)*。

**打印 RDD 中的元素：**

另外一个常见的问题是试图通过 *rdd.foreach(println)* 或 *rdd.map(println)* 来打印 RDD 中的元素。在一台机器上运行，这个会像预期一样输出 RDD 中的元素。然而在集群模式下，executor 调用 *stdout*，输出会写入 executor而不是 driver，因此 driver 的 *stdout* 不会显示输出信息。要在 driver 上输出 RDD 的元素，可以使用 *collect()* 方法把 RDD 收集到 driver 节点：*rdd.collect().foreach(println)*。

### 处理 K-V 对

大多数 Spark 操作可以作用在包含任何类型对象的 RDD 上，有一些指定的操作只能作用在 k-v 形式的 RDD 上。最常见的是 "shuffle" 操作，比如，group 或 aggregate 元素通过 key。在 Scala 中，对于包含元组的 RDD，这些操作是自动可选的。

下面的代码使用 *reduceByKey* 操作 k-v 对，统计文件中每行文本出现的次数：

```scala
val lines = sc.textFile("data.txt")
val pairs = lines.map(s => (s, 1))
val counts = pairs.reduceByKey((a, b) => a + b)
```

我们也可以使用 *counts.sortByKey()* 按字母顺序对 k-v 对进行排序。

### Transformations

下表中列出了一些常用的 transformations 操作：

| Transformation                                           | Meaning                                                      |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| map(func)                                                | 使用 *func* 对源数据中的每个元素进行处理，返回一个新的 RDD   |
| filter(func)                                             | 使用 *func* 对元数据中的每个元素进行处理，为 true 元素组成一个新的 RDD |
| flatMap(func)                                            | 跟 map 类似，但是每个输入项可以映射到 0 个或多个输出项(因此 *func* 应该返回一个序列) |
| mapPartitions(func)                                      | 跟 map 类似，但是在 RDD 的每个分区上单独运行，因此 *func* 必须是 Iterator\<T> => Iterator\<U> 类型的 |
| mapPartitionsWithIndex(func)                             | 跟 mapPartitions 类似，但是给 *func* 提供一个整数代表分区的索引，因此 *func* 必须是 (Int, Iterator\<T> => Iterator<U\>) 类型的 |
| sample(withReplacement, fraction, seed)                  | 使用改定的随机数生成 seed, 对数据中的一小部分进行取样        |
| union(otherDataset)                                      | 合并源数据和参数中指定的数据集，返回一个新的数据集           |
| intersection(otherDataset)                               | 取两数据集的交集，返回一个新的数据集                         |
| distinct([numPartitions])                                | 对源数据集去重后，返回一个新的数据集                         |
| groupByKey([numPartitions])                              | 当作用在 K-V 对数据集上时，返回一个 (K, Iterale<V\>) 格式的数据集。<br />如果是想通过 key 实现数据聚合(比如,sum 或 average)，使用 *reduceByKey* 或 *aggregateByKey* 会更好。<br />默认情况下，输出数据的并行度依赖于父 RDD 的分区数。可以通过 *numPartitions* 选项来设置 task 的个数。 |
| reduceByKey(func,[numPartitions])                        | 当作用在 (K,V) 对上时，返回一个 (K,V) 对格式的数据集，其中 V 是使用 *func* 对数据进行聚合的结果。reduce task 数可以通过第二个选项进行设置。 |
| aggregateByKey(zeroValue)(seqOp, combOp,[numPartitions]) | 当作用在 (K,V) 对上时，返回一个 (K,U)对格式的数据，使用给定的组合函数和中性的"0"值聚合每个键的值。允许跟输入类型不同的聚合类型，避免不必要的分配。reduce task 数可以通过第二个参数进行设置。 |
| sortByKey([ascending], [numPartitoins])                  | 当作用在 (K,V) 对上时，K 用来排序，返回一个通过键进行升序或降序排列的 (K,V)对格式的数据，*ascending* 选项的类型为布尔类型。 |
| join(otherDataset, [numPartitions])                      | 作用在 (K, V) 和 (K, W) 类型的数据上，返回 (K, (V, W)) 对格式的数据。可以通过 *leftOuterJoin*, *rightOuterJoin* 和 *fullOuterJoin* 进行外关联。 |
| cogroup(otherDataset, [numPartitions])                   | 作用在 (K, V) 和 (K, W) 类型的数据上，返回(K, (Iterable<V\>, Iterable<W\>)) 类型的数据，这个操作也叫做 *groupWith*。 |
| certesian(otherDataset)                                  | 笛卡尔积，作用在 T 和 U 类型的数据上，返回 (T, U) 类型的数据 |
| pipe(command, [envVars])                                 | 通过 shell 命令处理每个分区中的数据，RDD 的元素作为命令的标准输入，并从标准输出中输出 String 类型的 RDD |
| coalesce(numPartitions)                                  | 减少 RDD 的分区数到指定的数量，过滤大型数据集后，可以更有效地运行 |
| repartition(numPartitions)                               | 随机地重组数据，产生更多或更少的分区，这个操作会通过网络重新 shuffle 所有的数据。 |
| repartitionAndSortWithinPartition(partitioner)           | 根据给定的分割者对 RDD 进行重新分区，在新的分区中，根据 key 进行排序。这个比先调用*repartition* 操作，然后再每个分区中进行排序更高效 |

### Actions

下表中列出了一些常用的 actions 操作：

| Action                                   | Meaning                                                      |
| ---------------------------------------- | ------------------------------------------------------------ |
| reduce(func)                             | 使用 *func* (把两个数合成一个数的函数) 把数据集中的元素聚合。*func* 应该是可交换的和关联的，以便可以并行计算 |
| collect()                                | 已数组的形式返回数据集中的所有元素到 driver 程序。这个操作一般在数据集被过滤后或其它操作后，只保留了极小部分的子集后使用 |
| count()                                  | 返回数据集中元素的数量                                       |
| first()                                  | 返回数据集中的第一个元素 (跟 *take(1)* 类似)                 |
| take(n)                                  | 以数组的形式返回数据集的前 n 个元素                          |
| takeSample(withReplacement, num, [seed]) | 从数据集中随机去 num 个元素组成一个样本，并以数组的形式返回  |
| takeOrdered(n, [ordering])               | 去 RDD 中自然顺序或通过自定义排序器排序后的前 n 个元素       |
| saveAsTextFile(path)                     | 把数据集中的元素以文本的形式写入到本地文件系统、HDFS 或其它 Hadoop 支持的文件系统 |
| saveAsSequenceFile(path)                 | 把数据集中的元素写入到 Hadoop  序列化文件                    |
| saveAsObjectFile(path)                   | 以 Java 序列化格式写数据集中的元素，它可以通过 *SparkContext.objectFile()* 进行加载 |
| countByKey()                             | 只能用于 K-V 格式的数据，统计每个 key 的个数，返回一个 hashmap (K， int) 格式的数据 |
| foreach(func)                            | 把 *func* 函数作用于集合中的每个元素上。通常用于更新累加器或跟外部存储系统交互。<br />使用定义在 *foreach()* 外的变量会出现一些意想不到的结果 |

### Shuffle operations

某些操作会触发 Spark 的shuffle，shuffle 是 Spark 的一个机制，用来重新分配数据。shuffle 会造成数据在 executor 和节点之间复制，因此 shuffle 是一个复杂且高消耗的操作。

**背景：**

为了理解 shuffle 过程中发生了什么，可以以 *reduceByKey* 为示例。*reduceByKey* 操作会形成一个新的 RDD，同一个 key 的所有 value 聚合为一个值，这个值是对 key 对应的所有 value 执行 reduce 函数的结果。

在计算过程中，一个 task 对应一个分区，为了执行 *reduceByKey* 的reduce task 来组织所有的数据，Spark 必须读取所有分区上的所有数据，把不同分区上同一个 key 对应的 value 聚集在一起，最后得到每个 key 对应的聚合值，这个操作成为 shuffle。

虽然 shuffle 后的数据每个分区中的数据集是确定的，分区也是有序的，但是每个分区中的元素不是有序的。如果想要 shuffle 后分区中的元素也是有序的，可以使用下面的操作：

- *mapPartitions*, 排序每个分区中的数据，通过 *.sorted*
- *repartitionAndSortWithPartitions*, 在重新分区的同时进行排序
- *sortBy*, 形成一个全局有序的 RDD

会造成 shuffle 的操作有：

- repartition 操作，repartition 和  coalesce
- ByKey 操作，groupByKey 和 reduceByKey(除了 counting)
- join 操作，cogroup 和 join

**效率影响：**

Shuffle 是一个高消耗操作，它会引起磁盘 I/O、数据序列化和网络 I/O。为了组织 shuffle 的数据，Spark 会形成一系列的 map-task 来组织数据，形成一系列的 reduce-task 来聚合数据。map-task 和 reduce-task 这两个术语来自于 Hadoop MapReduce，跟 Spark 的 *map* 和 *reduce* 操作没有直接关系。

单个 map-task 的计算结果会保存在内存中，直到内存放不下。然后它们根据目标分区进行排序，并写入到单个文件。reduce-task 读取相应的排序块。

 某些 shuffle 操作会消耗大量的内存，因为采用保存在内存的数据结构来组织传输前后的数据。特别是，*reduceByKey* 和 *aggregateByKey* 操作在 map 端创建这些 数据结构，然后 *ByKey* 操作在 reduce 端生成它们的数据。当内存中放满时，数据会溢出到磁盘，导致额外的磁盘 I/O 并增加垃圾回收。

Shuffle 也会在磁盘上生成大量的中间文件，在 Spark 1.3，这些中间文件会一直保存直到 RDD不在被使用才会被垃圾回收。这样做的话，如果 lineage 被重新计算时，不需要再产生这些文件。如果应用程序保持对 RDD 的引用，或垃圾回收执行频率较低，垃圾回收可能会在很长一段时间后才会被触发，这也意味着长时间运行的 Spark job 会消耗大量的磁盘空间。临时保存目录在创建 SparkContext 时通过 *spark.local.dir* 参数指定。

可以通过各种参数来调整 shuffle 行为，具体配置可参照 [SparkConfiguration Guid](<http://spark.apache.org/docs/2.3.0/configuration.html>) 中的  *Shuffle Behavior*。

### RDD 持久化

Spark 最重要的一个特性是持久化或缓存数据集到内存。当持久化一个 RDD，每个节点都会存储 RDD 在内存中计算的部分，并在改数据集的其它 action 操作中重用它们。

可以通过 *persist()* 或 *cache()* 方法来持久化 RDD。第一执行 action 操作得到这个 RDD 时，它会保存在节点的内存中。Spark 的持久化是可以容错的，如果 RDD 的任何分区丢失，它会自动重算。

持久化 RDD 时可以设置不同的存储级别，可以持久化数据到磁盘，可以持久化到内存并序列化(节省空间)，存放它的副本在不同的节点。可以通过给 *persist()* 方法设置一个 *StorageLevel*  来设置存储级别。*cache()* 方法是简介版的持久化方法，它的默认存储级别是 *StorageLevel.MEMORY_ONLY* (存储非序列化对象到内存)。完整的存储级别入下表：

| Storage Level                            | Meaning                                                      |
| ---------------------------------------- | ------------------------------------------------------------ |
| MEMORY_ONLY                              | 以非序列化的对象的格式存储 RDD 到 JVM。如果内存放不下 RDD，一些分区不会被持久化，并且会在每次用到它时被重新计算。这是默认级别。 |
| MEMORY_AND_DISK                          | 以非序列化的对象的格式存储 RDD 到 JVM。如果内存放不下 RDD，把内存放不下的分区存放到磁盘。 |
| MEMORY_ONLY_SER<br />(Java and Scala)    | 以序列化对象(每个分区是一个字节数组)的格式存储 RDD，这个比非序列化对象更节省空间，但是读取数据的时候会消耗更多的 CPU。 |
| MEMORY_AND_DISK_SER<br/>(Java and Scala) |                                                              |
| DISK_ONLY                                | 存放 RDD 的分区在磁盘上                                      |
| MEMORY_ONLY_2, MEMORY_AND_DISK_2, etc.   | 存储分区数据在两个节点上                                     |
| OFF_HEAP(实验阶段)                       | 类似于 *MEMORY_ONLY_SER*，存储数据在堆外内存，要求堆外内存是可用的 |

> 在 Python  API 中，存储对象都会使用 *Pickle* 库进行序列化，所以是否选择序列化对它没有影响。Python API 中可选的存储级别有：*MEMORY_ONLY, MEMORY_ONLY_2, MEMORY_AND_DISK, MEMORY_AND_DISK_2, DISK_ONLY, and DISK_ONLY_2*。

Spark 也会持久化一些 shuffle 的中间数据(比如，reduceByKey)，甚至不需要使用 *persist*。这样做是为了避免 shuffle 过程中失败了重算整个输入数据。非常推荐使用 *persist* ，如果 RDD 计划被多次使用。

#### 选择哪个存储级别

Spark 的存储级别主要是权衡内存的占用和 CPU 效率。推荐使用下面的方法来选择：

- 如果 RDD 完全可以已默认存储级别 (MEMORY_ONLY) 存放在内存中，就使用这种方式进行存储。这个存储级别 CPU 效率最高，且 RDD 上的操作可以快速运行
- 第二个考虑使用 *MEMORY_ONLY_SER* 存储级别，并选择使用 fast serialization 库，这样更节省空间，并且处理速度也可以接受
- 不要把数据溢出到磁盘上，除非计算数据集的操作是非常高消耗的，或过滤大量的数据。否则，重新计算数据或许比从磁盘读取数据要快
- 如果想快速的恢复数据，可以设置副本。所有的存储级别都可以通过重算数据来提供容错，副本可以让你直接在丢失分区的 RDD 进行计算，不需要等待它被重算。

#### 移除数据

Spark 监控每个检点上缓存的使用情况，并以最近最少使用的方法移除旧数据的分区。也可以手动移动 RDD，通过 *RDD.unpersist()* 方法。

## 共享变量

通常当通过 Spark 操作来运行一个函数(比如，*map* 或 *reduce*)，是在远程的集群节点运行，但是函数中运行的变量都是单独拷贝的。变量会被复制到每台机器上，并且远程机器上更新的变量不会传播会 driver 程序。跨 task 读写分享变量是低效的。Spark 提供了两类 *shared variables*: 广播变量 (*broadcast variables*) 和累加器 (*accumulators*)。

### 广播变量

广播变量允许程序保存一个只读变量缓存在每台机器上，而不是在 task 中保存一个副本。它可以以高效的方式给每个节点一个大量数据输入的副本。Spark 还尝试使用高效的广播算法分发广播变量以降低通信成本。

Spark action 通过一系列的 stage 执行的，stage 是通过 shuffle 操作划分的。在每个 stage 中 Spark 会自动广播 task 中共同的数据。数据广播的方式是先以序列化的格式 cache，然后在 task 执行前进行反序列化。这意味着显示的创建广播变量只有当跨越多个 stage 的 task 需要相同的数据，或者通过非序列化格式 cache 数据是重要的。

广播变量创建和调用方法：

```scala
// 创建
scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar: org.apache.spark.broadcast.Broadcast[Array[Int]] = Broadcast(0)

// 调用
scala> broadcastVar.value
res0: Array[Int] = Array(1, 2, 3)
```

### 累加器

累加器通常用来实现计数器和 sum，Spark 本身就支持数字类型的累加器，开发者可以添加对其它类型的支持。

数字类型的累加器可以通过 *SparkContext.longAccumulator()* 或 *SparkContext.doubleAccumulator()* 来创建。运行在集群上的 task 可以使用 *add* 方法来加累加器，但是它们不能读取累加器的值，只有 driver 程序可以通过 *value* 方法来读取累加器的值。

一下代码展示了使用累加器来加和数组中的元素：

```scala
scala> val accum = sc.longAccumulator("My Accumulator")
accum: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 0, name: Some(My Accumulator), value: 0)

scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s

scala> accum.value
res2: Long = 10
```

这个示例中创建了一个 Long 类型的累加器，我们可以通过继承 *AccumulatorV2* 类来创建其它类型的累加器，需要重写 *AccumulatorV2* 中的一些方法：

- *reset*，这是累加器为 0
- *add*，加和另外一个值到累加器中
- *merge*，合并另一个相同类型的累加器

还有一些其它的方法，可以参考 [API documentation](http://spark.apache.org/docs/2.3.0/api/scala/index.html#org.apache.spark.util.AccumulatorV2)。示例：

```scala
class VectorAccumulatorV2 extends AccumulatorV2[MyVector, MyVector] {

  private val myVector: MyVector = MyVector.createZeroVector

  def reset(): Unit = {
    myVector.reset()
  }

  def add(v: MyVector): Unit = {
    myVector.add(v)
  }
  ...
}

// Then, create an Accumulator of this type:
val myVectorAcc = new VectorAccumulatorV2
// Then, register it into spark context:
sc.register(myVectorAcc, "MyVectorAcc1")
```

> Myvector 是一个已经写好的代表数学适量的类

累加器的值只有在 action 操作是才会更新，Spark 保证每个 task 只会更新累加器一次，比如，重启 task 不会更新累加器 的值。对于 transformations 操作，开发者 必须认识到如果 stage 被重新执行，每个 task 可能会多次更新累加器。

累加器也是懒惰计算，如果累计器是在 transformations 操作中更新，它的值也只会在执行 action 操作是更新。

# 集群模式概览

这部分内容主要介绍 Spark 如何运行在集群上，便于理解 Spark 包含的各种组件。

## 组件

Spark 应用程序以一系列的进程运行在集群上，通过主程序( driver program) 上的 *SparkContext* 调用。

在集群上运行，SparkContext 可以连接多种集群管理(Spark standalone 集群管理, Mesos 或 YARN)，它们给应用程序分配资源。一旦连接，Spark 从集群节点请求*executor*，executor 用来运行和存储程序的数据。然后，发送应用程序代码(由 jar 或 Python 文件定义的代码，通过 SparkContext 发送) 到 executors。最后 SparkContext 发送 tasks 到 executor 运行。

![](http://spark.apache.org/docs/2.3.0/img/cluster-overview.png)

简要介绍下这种结构：

1. 每个应用程序获取它们自己的 executor 进程，这些进程会伴随整个程序，并且在多个线程中运行 task。这样对格力应用程序非常有益，在安排任务这一边(每个 driver 安排它们自己的 task)，在 executor 这一边(不同应用程序的 task 运行在不同的 JVM 上)。这也意味着如果不把数据写入外部存储系统，数据就不能再不同的 Spark 应用程序之间共享。
2. Spark 跟底层的集群管理器无关的，只要它可以获取到 executors 进程，并且这些进程之间是可以相互通信的，即使集群管理器也支持其它应用程序，它也可以运行(比如, Mesos 或 YARN)。
3. driver 程序必须监听和接受来自 executor 的连接。
4. 因为 driver 程序安排集群上的 tasks, 所以它应该运行在离 worker 节点较近的地方，优选同一个网络内的机器。如果你想远程发送请求到集群，最好给 driver 开启一个 RPC ，并让它从附近节点提交操作，而不是与远离 worker 节点。

## 监控

每个 driver 程序都有一个 Web UI，一般是在 4040 端口，这里展示 tasks, executors 和 存储空间使用信息。可以通过在浏览器访问 *http://\<driver-node>:4040* 来打开 WebUI。具体 monitor 介绍在 [monitoring guide](http://spark.apache.org/docs/2.3.0/monitoring.html)。

## 术语

下表总结了常用的概念：

| Term            | Meaning                                                      |
| --------------- | ------------------------------------------------------------ |
| Application     | 用户构建在 Spark 上的程序。由集群上的 driver 程序和 executors 组成 |
| Application jar | 包含用户 Spark 应用程序的 jar。用户的 jar 包中不应该包含 Hadoop 和 Spark 相关的库，这些库在运行时已经包含了 |
| Driver program  | 运行应用程序的 main 函数，并创建 SparkContext                |
| Cluster manager | 一个外部服务用来从集群上请求资源，比如，standalone, Mesos, YARN |
| Deploy mode     | 区分 driver 程序在哪里运行。"cluster" 模式，在集群内加载 driver 程序。"client" 模式，在集群外运行 driver |
| Worker node     | 集群中任何可以运行程序的节点                                 |
| Executor        | 一个运行在 worker 节点上的进程，用来加载程序，它可以运行 task，并在内存和磁盘保存数据。每个程序都运行在自己的 executor 上 |
| Task            | 发送 executor 的一个工作单元                                 |
| Job             | 由多个 task 组成的并行计算，这些 task 是对 Spark action 的相应(比如，save, collect)。可以通过 driver 的 log 来查看 |
| Stage           | 每个 Job 可以被划分为更小的 task 集合，这个 task 集合就叫 stage, 每个 stage 之间相互依赖(跟 MapReduce 中的 map 和 reduce stage 类似)，可以通过 driver 日志查看 |





# 提交应用程序

*SPARK_HOME/bin/spark-submit* 脚本是用来提交应用程序到集群。可以通过 sbt 或 Maven 把应用程序打成 jar 包，jar 中 Spark 和 Hadoop 相关的依赖可以设置为 `provided` 级别，因为在运行程序时，集群管理已经提供了这些依赖。

## spark-submit 脚本

命令格式：

```shell
./bin/spark-submit \
 --class <main-class> \
 --master <master-url> \
 --deploy-mode <deploy-mode> \
 --conf <key>=<value> \
 ... # other options
 <application-jar> \
 [application-arguments]
```

> 常用选项说明：
>
> - --class: 应用程序的全路径 (比如，org.apache.spark.examples.SparkPi)
> - --master: 指定集群的 master URL (比如：spark://23.195.26.187:7077)
> - --deploy-mode: cluster, 部署 driver 端到 worker 节点上；client (默认选项), 本地作为一个外部客户端。
> - --conf: 通过 k-v 形式，指定 Spark 配置属性，如果属性值中包含空格，可以通过双引号包含，比如：“key=value"。
> - application-jar: 包含应用程序和依赖度的 jar 包路径，指定这个 URL 必须是集群可以访问的路径。比如，可以指定一个 hdfs:// 或 file:// 路径，这个路径是在所有节点都可以访问的。
> - 传递给主类 main 方法的参数。

一个常用的部署策略是通过一个跟 work 节点协同工作的机器来提交应用程序(比如，一个 standalone 模式集群的 Master 节点)。这种方式非常适合使用 *client* 模式，driver 程序直接在 spark-submit 进程中启动，该进程充当集群的客户端，应用程序的输入和输出都会附加到控制台。因此这个模式非常适合涉及到交互的应用程序。

如果应用程序从一台远离 worker 节点的机器提交(比如，本地机器)，通常使用 *cluster* 模式来最小化 driver 端和 executor 之间的网络延迟。目前，*standalone* 模式不支持 Python 应用程序的 *cluster* 模式。

对于 Python 应用程序，只需要把 \<application-jar> 参数的 jar 包换成 .py 文件，并通过 --py-files 参数把 .zip/.egg/.py 文件添加到搜索路径。

对于一些特定的集群管理模式，还有一些特定的选项可以使用。比如，在 Spark standalone 集群中使用 *cluster* 部署模式，可以指定 *--supervise* 参数来确保 driver 端失败后重启。可以通过 *spark-submit --help* 命令来查看具体的选项。一下是一些示例：

```shell
# 在本地运行，分配 8 核
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[8] \
  /path/to/examples.jar \
  100

# 在 Spark standlone 集群以 client 部署模式运行
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# 在 Spark standlone 集群以 cluster 部署模式运行，并带有 supervise 选项
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --deploy-mode cluster \
  --supervise \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# 在 YARN 集群运行
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode cluster \  # can be client for client mode
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000

# 在 Spark standalone 集群运行 Python 程序
./bin/spark-submit \
  --master spark://207.184.161.138:7077 \
  examples/src/main/python/pi.py \
  1000

# 在 Mesos 集群以 cluster 部署模式运行，并带有 supervise 选项
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master mesos://207.184.161.138:7077 \
  --deploy-mode cluster \
  --supervise \
  --executor-memory 20G \
  --total-executor-cores 100 \
  http://path/to/examples.jar \
  1000
```

## Master URLs

传递给 Spark 的 master URL 可以是一下格式：

| Master URL                      | 含义                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| local                           | 在本地以一个 worker 线程运行 Spark (根本没有并行性)          |
| local[K]                        | 在本地以 K 个 worker 线程运行 Spark (理论上，设置为机器的核数) |
| local[K,F]                      | 在本地以 K 个 worker 线程运行 Spark，并设置 task 最大失败次数为 F |
| local[*]                        | 在本地运行 Spark，worker 的线程数跟机器的洛基核个数一致      |
| local[*,F]                      | 在本地运行 Spark，worker 的线程数跟机器的洛基核个数一致，并设置 task 最大失败次数为 F |
| spark://HOST:PORT               | 连接到给定的 Spark standalone 集群的 master，这个端口必须是 master 配置使用的端口，默认为 7077 |
| spark://HOST1:PORT1,HOST2:PORT2 | 连接到通过 Zookeeper 配置了主备的 Spark standalone 集群。这个列表中必须包括高可用中配置的所有 master 的主机名。端口默认为 7077 |
| mesos://HOST:PORT               | 连接到 Mesos 集群。端口默认为 5050                           |
| yarn                            | 通过 *client* 或 *cluster* 部署模式连接到 yarn 集群。集群的位置会通过 *HADOOP_CONF-DIR* 或 *YARN_CONF_DIR* 变量来找到 |

## 从文件中加载配置

*spark-submit* 脚本可以从配置文件加载默认 Spark 配置到应用程序。默认情况下，会从 *conf/spark-default.conf* 文件读取配置。

使用默认的 Spark 配置，可以在使用 spark-submit 脚本时省略一些选项，比如，如果配置文件中设置了 *spark.master*  属性，可以在使用 spark-submit 脚本是省略 *--master* 选项。通常情况下，通过 SparkConf 的配置有最好的优先级，然后是在 *spark-submit* 脚本中指定的，最后是默认配置文件中的。

如果你不清楚配置项是从哪里来的，可以在运行 *spark-submit* 脚本是加上 *--verbose* 选项，来输出 debug 信息。

## 依赖管理

当使用 spark-submit 脚本，应用程序的 jar 包依赖的任何 jar 包，都可以通过 *--jars* 选项来上传到集群，如果依赖的是多个 jar 包，可以通过逗号分隔。jar 包列表会被包含在 driver 和 excutor 类路径中。

在 Spark 中传递 jar 包，可以使用以下的 URL 方案：

- file: -绝对路径 和 file:/ URIs，由 driver 的 HTTP 文件服务提供，并且每个 executor 从 driver 的 HTTP 服务拉取文件。
- hdfs:, http:, https:, ftp: 这些是从 URI 中拉取文件和 jar 包
- local: 一个以 local: 开始的 URI, 以本地文件的形式存在于每个 worker 节点上。这意味着不会有网络 IO，大的文件或 jar 包，会被发送到每个 worker 节点，或通过 NFS，GlusterFS 进行分享

jar 包和文件会被复制到 executor 节点的每个 SparkContext 的工作目录。随着时间推移，这个会占用许多空间，并且需要并清理。如果使用 YARN，会自动清理，如果使用的 Spark standalone，可以通过配置 *spark.worker.cleanup.appDataTtl* 属性来设置自动清理。

用户可以通过 *--packages* 指定 Maven 坐标列表(以逗号分隔)来引入其他依赖。使用这个命令时，会处理所有的依赖项。其它的仓库(比如：SBT)，可以通过 *--responsitories* 参数来引入。

对于 Python，*--py-files* 选项可以用来分配 .egg, .zip 和 .py 文件到 executor。

# 集群模式

Spark 可以自己运行，也可以通过几个现有的集群管理器运行。它目前支持一下几种调度模式:

- Standalone
- Apache Mesos
- Hadoop Yarn
- Kubernetes

## Spark Standalone Mode

Spark 提供了一个独立运行的部署模式，也可以手动开启一个 master 和 workers 来手动加载一个 standalone 集群，或者通过已经提供好的脚本。也可以在一台机器上运行这些进程。

### 手动启动集群

启动 master:

```shell
./sbin/start-master.sh
```

启动后，master 会输出 *spark://HOST:PORT* URL，可以通过这个连接 workers，也可以作为*SparkContext* 的 *master* 参数的值。也可以在 master 的 web UI 上找到这个地址，master 的 Web UI 默认地址是 *http://localhost:8080*。

启动 worker:

```shell
./sbin/start-slave.sh <master-spark-URL>
```

启动 worker 节点后，查看 master 的 WebUI(默认 http://localhost:8080)，就会看到新的节点列表，还有 CPU 个数和内存。

下面这些参数可以传值该 master 和 worker：

| argument                | meaning                                                      |
| ----------------------- | ------------------------------------------------------------ |
| -h Host, --host Host    | 监听的主机                                                   |
| -i Host, --ip Host      | 监听的主机(已启用，可以使用 -h 或 --host)                    |
| -p Port, --port Port    | 监听服务的端口(默认，master 是 7077, worker 是随机的)        |
| --webui -port PORT      | web UI 的端口(默认，master 8088, worker 8081)                |
| -c Cores, --cores cores | 节点上运行的 Spark 程序可以使用的 CPU 核数，只有 worker 可以设置，默认所有都可用 |
| -m Mem, --memory Mem    | 节点上运行的 Spark 程序可以使用的总内存大小，只有 worker 可以设置，默认总内存减 1GB |
| -d Dir, --worke-dir Dir | 用于临时空间和作业输出日志的目录，只有 worker 可以设置，默认 SPARK_HOME/work |
| --properties-file FILE  | 用于加载 Spark 属性文件的路径，默认 conf/spark-default.conf  |

### 集群启动脚本

通过启动脚本启动 Spark standalone 集群，需要再 conf/ 目录下创建一个 slaves 文件，这个文件中需要包含所有 worker 节点的 hostname，一行写一个。如果没有 slaves 文件，启动脚本默认认为这是一个单节点机器。master 节点可以通过 ssh 来访问 worker 节点，ssh 需要并行运行，并需要配置免密登录。如果没有设置免密登录，可以设置 SPARK_SSH_FOREGROUND 环境变量，并给每个 worker 节点设置密码。

设置了 slaves 文件后，就可以通过下面的脚本来启动和停止集群，可以使用 Hadoop 的脚本，也可以使用 SPARK_HOME/sbin下的脚本：

- start-master.sh, 启动 master 实例
- start-slaves.sh, 启动 slaves 文件制定的机器上的 slave 实例
- start-slave.sh, 启动运行脚本这台机器上的 slave 实例
- start-all.sh, 启动 master 和所有的 slave 实例
- stop-master.sh, 停止 master
- stop-slaves.sh, 停止 slaves 文件制定的机器上的 slave
- stop-slave.sh, 停止运行脚本这台机器上的 slave
- stop-all.sh, 停止 master 和所有的 slave

> 这些脚本必须在你想运行 Spark master 的机器上运行，而不是你本地机器

你可以通过在 *conf/spark-env.sh* 中设置环境变量来进一步配置集群，可以通过 *conf/spark-env.sh.template* 文件来创建这个文件，还需要把这个文件复制到所有的 worker ，配置才会生效。下面是可以设置的变量：

| 环境变量                | 意义                                                         |
| ----------------------- | ------------------------------------------------------------ |
| SPARK_MASTER_HOST       | 将 master 绑定到特定的主机或 ip 地址，比如公共主机           |
| SPARK_MASTER_PORT       | 在不同的端口启动 master                                      |
| SPARK_MASTER_WEBUI_PORT | master 的 WebUI 端口(默认：8080)                             |
| SPARK_MASTER_OPTS       | 设置仅作用在 master 的属性，以 “-Dx=y" 的格式                |
| SPARK_LOCAL_DIRS        | Spark 的临时目录，保存 map 输出文件和需要保存在磁盘的 RDD。这个目录应该是系统中的快速磁盘。可以通过逗号分隔自定不同磁盘上的多个目录 |
| SPARK_WORKER_CORES      | Spark 程序可以使用的 CPU 核数                                |
| SPARK_WORKER_MEMORY     | Spark 程序可以使用的内存。可以通过 spark.executor.memory 指定每个 Spark 程序使用的内存 |
| SPARK_WORKER_PORT       | 在指定的端口启动 Spark worker，默认是随机端口                |
| SPARK_WORKER_WEBUI_PORT | worker WebUI 使用的端口                                      |
| SPARK_WORKER_DIR        | 用来运行程序的目录，包含日志和临时空间，默认：SPARK_HOME/work |
| SPARK_WORKER_OPTS       | 设置作用在 worker 上的属性，以"-Dx=y"的格式                  |
| SPARK_DAEMON_MEMORY     | Spark master 和 worker 进程使用的内存数，默认 1G             |
| SPARK_DAEMON_JAVA_OPTS  | 以 "-Dx=y" 的格式设置 master 和worker 进程的 JVM 属性        |
| SPARK_DAEMON_CLASSPATH  | spark master 和 worker 进程的 classpath                      |
| SPARK_PUBLIC_DNS        | Spark master 和 worker 的公共 DNS                            |

> spark 启动脚本不支持 Windows, 在 Windows 上运行 Spark，可以手动启动 master 和 worker

SPARK_MASTER_OPTS 支持以下属性：

| Property Name                     | Default    | Meaning                                                      |
| --------------------------------- | ---------- | ------------------------------------------------------------ |
| spark.deploy.retainedApplications | 200        | 展示已完成程序的最大数量，更旧的程序会从 UI 中删除           |
| spark.deploy.retainedDrivers      | 200        | 展示已完成 driver 的最大数量                                 |
| spark.deploy.spreadOut            | true       | standalone 集群是跨节点扩展应用程序还是尝试将他们合并到尽可能少的节点上。如果数据在 HDFS 上扩展出去会比较好，对于计算密集型工作，聚集在一起会比较好 |
| spark.deploy.defaultCores         | (infinite) | 如果没有设定 spark.cores.max, 通过这个来指定分配给应用程序的 CPU 核数。如果没有设置这个参数，也没有设置 *spark.cores.max* 参数，会分配给应用程序所有可用的 CPU 核。在共享集群上，把这个值设置的低一些，防止程序获取整个集群的 CPU 核 |
| spark.deploy.maxExecutorRetries   | 10         | 设置集群管理员移除失败程序前的重试次数。任何应用程序，如果有正在运行的 executor,是不会被移除的。如果要禁用自动删除，可以设置参数为 -1 |
| spark.worker.timeout              | 60         | master 接受不到 worker 的心跳时，认为 worker 丢失的秒数      |

SPARK_WORKER_OPTS 支持以下属性：

| Property Name                                    | Default          | Meaning                                                      |
| ------------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| spark.worker.cleanup.enable                      | false            | 周期性清除 worker/application 目录。这个只对 standalone 有用，YARN worker 不同。只有已经停止的 application 目录会被清除 |
| spark.worker.cleanup.interval                    | 1800(30 minutes) | 设置清除的周期，单位 s,                                      |
| spark.worker.cleanup.appDataTtl                  | 604800 (7 days)  | 保持 application 工作目录的时长，单位 s。这个时间应该由磁盘的空间来决定。工作目录很快会被放满，尤其是非常频繁运行 job 时 |
| spark.worker.ui.compressedLogFileLengthCacheSize | 100              | 对于已经压缩的日志文件，只能通过解压缩来计算未压缩文件。Spark 缓存已经压缩日志文件的解压缩文件大小。这个属性用来设置缓存的大小 |

### 连接应用程序到集群

在 Spark 集群运行一个应用程序，可以在创建 SparkContext 时指定 master 为 *spark://IP:PORT*。

在集群运行 Spark shell，可以使用下面的命令：

```shell
./bin/spark-shell --master spark://IP:PORT
```

> 也可以使用 *--total-executor-cores <numCores>*  来指定 spark-shell 在集群上使用的 CPU 核数

### 加载 Spark 应用程序

*spark-submit* 脚本可以提价应用程序到集群，对于 standalone 集群，Spark 支持两种部署模式：

- client, deriver 跟提交应用程序的客户端在同一个进程中
- cluster, driver 在集群中的 worker 节点上启动，并且 client 进程一旦完成它提交应用程序的责任就会推出，不会等待应用程序执行完

如果应用程序时通过 Spark submit 提交的，应用程序的 jar 包会自动发散到各个 worker 节点上。对于应用程序依赖度的 jar 包，可以通过 --jars 参数来指定(通过逗号分隔，比如，--jars jar1,jar2)。

standalone 的 *cluster* 模式可以在应用程序的退出码不为 0 时，重新启动应用程度。要使用这个特性，可以在通过 *spark-submit* 提交应用程序时指定 *--supervise* 参数。如果你想干掉一个一直在失败的应用程序，可以通过下面的 命令：

```shell
./bin/spark-class org.apache.spark.deploy.Client kill <master url> <driver ID>
```

> 可以通过 standalone 的 Master 的 Web UI 来找到 driver ID, Web UI 的地址为 http://<master url>:8080

### 资源调度

standalone 集群目前只支持 FIFO 资源调度，如果要多个用户同时使用，可以通过控制每个应用程序使用的最大资源数。默认，每个应用程序可以使用集群的所有 CPU 核，这也意味着同一时间只能运行一个程序。可以在 SparkConf 中通过 *spark.cores.max* 来指定所使用的 CPU 核数:

```scala
val conf = new SparkConf()
  .setMaster(...)
  .setAppName(...)
  .set("spark.cores.max", "10")
val sc = new SparkContext(conf)
```

也可以配置 master 进程的 *spark.deploy.defaultCores* 选项来改变应用程序的默认配置，以改变没有设置 *spark.core.max* 应用程序的默认值。可以在 conf/spark-env.sh 文件中添加：

```shell
export SPARK_MASTER_OPTS="-Dspark.deploy.defaultCores=<value"
```

### Executors 调度

分配给每个 executor 的 CPU 核是可以设置的，如果 *spark.executor.cores* 是被明确指定的，如果 worker 节点有足够的 cores 和 内存，那么一个应用程序的 executor 会被分配到同一个 worker 节点上，如果没有指定，那么每个 executor 默认获取 worker 节点上所有可用的 cores，这样每个 worker 节点只能加载一个应用程序的一个 executor。

### Monitor && Logging

Spark 的 standalone 模式提供了一个基于 Web 的用户界面来监控集群。master 和每个 worker 都有它们自己的 web UI，展示集群和 job 的统计信息。默认情况下可以通过 8080 端口登录 master 的 web UI，这个端口可以通过配置文件或命令行修改。

每个 job 的详细日志会写到每个 slave 节点的工作目录(默认为 SAPRK_HOME/work)。还可以看到两个文件：stdout 和 stderr, 所有输出会写入到控制台。

### 跟 Hadoop 一起运行

如果 Spark 作为一个单独的服务跟 Hadoop 运行在相同的机器上，可以使用 hdfs://URL(hdfs://\<namenode>:9000/path，URL 可以在 Hadoop 的 Namenode 的 Web UI 发现) 来通过 Spark 访问 Hadoop 上的数据。另外，你可以设置一个单独的 Spark 集群，并还是可以通过网络访问 HDFS 的数据，这个会被访问本地磁盘慢。

### 高可用

standalone 集群对失败的 worker 是可以快速恢复的(把失败的 worker 移动到其它 worker 上)。但是 master 是用来做资源调度的，这个存在单点故障: 如果 master 挂掉后，没有新的 master 可以创建，为了解决这个问题，提供了一下两个高可用方案：

**通过 Zookeeper 设置备用 Master**

利用 Zookeeper 来提供 leader 选举并存储一些状态，可以在集群中加载多个连接在同一个 Zookeeper 的 Master。其中一个会被选为 "leader"，其它的会处于 standby 模式。如果当前的 leader 挂了，另外一个 Master 会被选举出来，重新这只旧的 Master 的状态，并回复调度。整个恢复过程会消耗 1 到 2 分钟。这个调度的延期只是对新的应用程序有影响，对在 Master 失败时已经运行的程序没有影响。

可以通过 spark-env 中的 *spark.deploy.recoverMode* 和 *spark.deploy.zookeeper.\** 配置 SPARK_DAEMON_JAVA_OPTS, 来开启这个模式。更多的配置说明可以参考 [configuration doc](https://spark.apache.org/docs/2.3.0/configuration.html#deploy)。

> 如果在集群中有多个 master, 但是没有通过 Zookeeper 配置 master, master 不会发现它们彼此，并认为它们都是 leader，这样集群中所有的 master 都是彼此独立的，集群是不会处于健康状态的。

如果已经拥有一个 Zookeeper 集群，那么配置高可用是非常简单的，通过相同的 Zookeeper 配置(ZK 的 URL 和目录) 在不同的节点来开启多个 Master 进程，Master 可以再任何时候添加和移除。

在调度应用程序和添加 worker 到集群时，它们需要知道当前 leader 的 IP 地址。可以穿入 master 地址的列表。比如，可以在 SparkContext 中设置 *spark://host1:port1,host2:port2*，这样如果 host1 挂了，SparkContext 会去发现新的 leader。

"registering with a Master" 跟正常操作之间存在一个重要不同点，在启动的时候，应用程序或 worker 需要找到并注册到当前的 leader master 上。如果注册成功，它会有一个 "in the system" 状态(存储在 Zookeeper中)。如果出现失败情况，新的 leader 会联系所有当前已经注册的应用程序和 worker, 通知它们 leader 已经改变，它们在启动时甚至不需要知道新 master 的存在。

因为这个属性，新的 Master 可以在任何时候被创建，我们只需要关心新的应用程序和 worker 可以被注册。

**通过本地文件进行单节点恢复**

Zookeeper 是生产环境进行高可用的最好方式，如果你只是想在 Master 挂掉后重启它，可以使用 FILESYSTEM 模式。在应用程序和 worker 注册时，会把状态信息写入指定目录，以便在重启 Master 进程时恢复它们。

可以通过 spark-env 的 SPARK_DAEMON_JAVA_OPTS 配置项来配置这个模式：

| System property               | Meaning                                            |
| ----------------------------- | -------------------------------------------------- |
| spark.deploy.recoverMode      | 默认为 NONE，设置为 FILESYSEM 来开启单节点恢复模式 |
| spark.deploy.recoverDirectory | 用来存储状态的目录，要求是 Master 可访问的         |

> - 这个解决方案可以跟过程监控/管理器一起使用，或仅仅允许通过重启来手动恢复
> - 通过文件系统恢复显然比什么都不做要好，这个模式可能是次最优的在一些开发或实验目的下。通过 *stop-master.sh* 脚本干掉 master 并不会清除 recover 状态，所以不管你什么时候开启一个新的 Master, 它都会进入 recover 模式，这个会增加 1 分钟的启动时间。
> - 虽然没有得到官方支持，你可以挂在一个 NFS 目录来作为恢复目录。

## Spark on YARN

从 Spark 0.6.0 版本开始支持 Spark on YARN 模式，并在随后的版本中得到改进。

### 配置 Spark on YARN

确保 HADOOP_CONF_DIR 或 YARN_CONF_DIR 指向一个包含 Hadoop 集群配置文件的目录。这些配置是用来往 HDFS 写入数据，并连接 YARN 的 ResourceManager。这个目录中包含的配置会被分发到 YARN 集群，以便所有应用程序使用的 container 使用相同的配置文件。如果引用 Java 系统属性或环境变量没有被 YARN 管理，那么它们应该在 Spark 应用程序中设置(当运行在 client 模式时，在 driver, executors 和 AM 中设置)。

通过 YARN 加载应用程序时有两种部署模式：

- cluster 模式，Spark driver 运行在 YARN 集群的 AM 进程上，并且 client 会在程序开始后关掉。
- client 模式，driver 运行在 client 进程中，AM 只是用来从 YARN 上获取资源

不像其它 Spark 支持的集群管理，master 的地址通过 *--master* 参数来指定，在 on YARN 模式中，ResourceManager 的地址从 Hadoop 配置文件中获取，因此 *--master* 参数应该指定为 *yarn*。

通过 cluster 模式加载 Spark 应用程序：

```shell
$ ./bin/spark-submit --class path.to.your.Class --master yarn --deploy-mode cluster [options] <app jar> [app options]
```

示例：

```shell
$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    --queue thequeue \
    examples/jars/spark-examples*.jar \
    10
```

在这个示例中启动了一个 YARN client 程序来启动默认的 Application Master，SparkPi 作为 Application Master 的子进程运行。client 会定期的拉取 Application Master 的更新状态，并展示在控制台。在应用程序运行结束的时候，client 会退出。

以 *client* 模式加载 Spark 应用程序，只需要把 *cluster* 替换为 *client*。下面展示了以 *client* 模式运行 spark-shell:

```shell
$ ./bin/spark-shell --master yarn --deploy-mode client
```

### 添加其它的 JAR 包

在 *cluster* 模式中，driver 跟 client 运行在不同的机器上，因此 *SparkContext.addJar* 不能使用位于 client 的文件。要让位于 client 的文件可以通过 *SparkContext.addJar* 使用，可以通过 *--jars* 来包含这些文件。

```shell
$ ./bin/spark-submit --class my.main.Class \
    --master yarn \
    --deploy-mode cluster \
    --jars my-other-jar.jar,my-other-other-jar.jar \
    my-main-jar.jar \
    app_arg1 app_arg2
```

### 准备

要在 YARN 上运行 Spark, 需要一个支持 YARN 的 Spark 二进制包，二进制包可以从[下载页面](http://spark.apache.org/downloads.html) 进行下载。

为了让 YARN 可以访问 Spark 运行时 jar 包，可以指定 *spark.yarn.archive* 或 *spark.yarn.jars* 参数。详细的配合信息可以参考 [Spark Properties](http://spark.apache.org/docs/2.3.0/running-on-yarn.html#spark-properties)。如果 *spark.yarn.archive* 和 *spark.yarn.jars* 都没有被指定，Spark 会在 $SPARK_HOME/jars 目录下创建包含所有 jar 包的 zip 文件，并上传它们到分布式缓存中。

### Debugging 应用程序

在 YARN 中，executor 和 application master 运行在 container 中，YARN 有两种模式来处理已经运行完 application 的 container 日志。如果开启了日志聚合(*yarn.log-aggregation-enable* 配置)，container 日志会复制到 HDFS，并从本地磁盘删除。这些日志可以通过 *yarn logs* 命令在集群的任何节点访问：

```shell
yarn logs -applicationId <app ID>
```

> 这个命令会数据指定应用程序在所有 containers 的日志

也可以直接使用 HDFS shell 或 API 查看 container 日志文件，日志文件的位置可以通过 YARN 的 *yarn.nodemanager.remote-app-log-dir* 和 *yarn.nodemanager.remote-app-log-dir-suffix* 配置来查看。日志也可以通过 Spark Web UI 下的 Executors 菜单页查看，需要开启 Spark history 和 MapReduce history 服务并配置 yarn-site.xml 文件中的 *yarn.log.server.url* 属性。Spark history 服务 UI 中的日志 URL 会重定向到 MapReduce history 服务中来展示聚合日志。

如果没有开启聚合日志，日志会保存在本地的 *YARN_APP_LOGS_DIR*，这个通常会配置为 */tmp/logs* 或 */HADOOP_HOME/logs/user*，这个跟 Hadoop 的版本有关。可以在运行 container 的主机的指定目录下查看 continer 的日志。子目录通过 application ID 和 container ID 来组织日志，日志也可以在 Spark Web UI 的 Executor 菜单下查看，并不需要开启 MapReduce history 服务。

增加 *yarn.nodemanager.delete.debug-delay-sec* 到一个更大的值(比如，3600)，并通过 *yarn.nodemanager.local-dirs* 配置允许应用程序缓存在 container 运行的节点。这个目录包含加载一个 container 需要的脚本、jars和环境变量。这些对调试应用程序问题是非常有用的。

自定义应用程序 master 和 executor 的 log4j 配置，一下是一些选项：

- 通过 spark-submit 上传 log4j.properties 文件，通过 *--files* 选项添加要上传的文件
- 添加 *-Dlog4j.configuration=\<location of configuration file> 到 Driver 的 *spark.driver.extraJavaOptions*  或 Executor 的 *spark.executor.extraJavaOptions*。如果使用的是一个文件，`file:` 需要被指定，并且文件需要存在于所有节点的本地目录。
- 更新 *$SPARK_CONF_DIR/log4j.properties* 文件，它会自动通过其它配置上传，其它两个配置方式比这种配置方式由更高的优先级。

> 第一种配置方式，每个  executor 和应用程序的 master 会分享相同的 log4j 配置，如果 master 和 executor 在同一个节点是会出现问题，比如，会把日志写入相同的日志文件

如果需要将日志放在 YARN，以便 YARN 可以展示和聚合日志，可以使用配置 log4j.properties 文件的 *spark.yarn.app.container.log.dir*，比如：*log4j.appender.file_appender.File=${spark.yarn.app.container.log.dir}/spark.log* 。对于流式应用程序，设置 *RollingFileAppender* 并设置日志文件位置为 YARN 的日志目录，可以避免磁盘爆满。

为应用程序的 master 和 executor 定义 metrics.properties，更新 *$SPARK_CONF_DIR/metrics.properties* 文件。这个文件会自动通过其它配置上传，不需要通过 *--files* 参数指定。

Spark 属性：

| Property Name                                     | Default                                      | Meaning                                                      |
| ------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| spark.yarn.am.memory                              | 512m                                         | YARN Application Master 在 client 模式时使用的内存，在 cluster 模式时使用 *spark.driver.memory* 来指定内存 |
| spark.yarn.am.cores                               | 1                                            | client 模式时 YARN Application Master 使用的核数，在 cluster 模式可以使用 *spark.driver.cores* 来 指定 |
| spark.yarn.am.waitTime                            | 100s                                         | 在 cluster 模式，YARN Application Master 等待 SparkContext 初始化的时间。在 client 模式，YARN Application Master 等待 driver 连接的时间 |
| spark.yarn.submit.file.replication                | HDFS 默认的副本数(3)                         | 应用程序上传到 HDFS 的文件的备份数，包含 Spark jar,app jar 和分布式缓存 |
| spark.yarn.stagingDir                             | 当前用户的 home 目录                         | 提交应用程序时使用的 staging 目录                            |
| spark.yarn.preserve.staging.files                 | false                                        | 设置为 *true* 会在程序运行结束后保存 staged 文件(Spark jar, app jar, 分布式环境文件)，而不是删除它们 |
| spark.yarn.scheduler.heartbeat.interval-ms        | 3000                                         | Spark application 跟 YARN ResourceManager 进行心跳的时间间隔，单位 ms，最大值为  YARN 满期配置的一半，比如，*yarn.am.liveness-monitor.expiry-interval-ms* |
| spark.yarn.scheduler.initial-allocation.interval  | 200ms                                        | 当存在待处理容器分配请求时，Spark application master 跟 YARN ResourceManager 进行心跳的时间间隔。它不应该比 *spark.yarn.scheduler.heartbeat.interval-ms* 大。如果依然存在待分配容器，心跳间隔会在原来基础上加倍，直到达到 *spark.yarn.scheduler.heartbeat.interval-ms* |
| spark.yarn.max.executor.failures                  | numExecutor *2, 最小值为 3                   | application 失败之前，最大失败 executor 次数                 |
| spark.yarn.historyServer.address                  | none                                         | Spark history 服务的地址，比如，*host.com:18080*，这个地址不应该包含协议(http://)，默认是没设置的。当 Spark 应用程序完成后，将应该用程序从 ResourceManager UI 链接到 Spark history UI，并将此地址提供该 YARN ResourceManager。对于 这个属性，可以使用 YARN 属性进行配置，并它们会在 Spark 运行时被替代。比如，如果 Spark histroy 服务跟  YARN ResourceManager 运行在同一台机器上，可以设置为 ${hadoopconf-yarn.resourcemanager.hostname}:18080 |
| spark.yarn.dist.archives                          | none                                         | 逗号分隔的存档列表，会被上传到每个 executor 的工作目录       |
| spark.yarn.dist.files                             | none                                         | 逗号分隔的文件列表，会被放置在每个 executor 的工作目录       |
| spark.yarn.dist.jars                              | none                                         | 逗号分隔的 jar 包列表，会被放置在每个 executor 的工作目录    |
| spark.yarn.dist.forceDownloadSchemes              | none                                         | 逗号分隔的方案列表，在将文件添加到 YARN 的分布式缓存之前将文件下载到本地磁盘。 |
| spark.yarn.am.memoryOvehead                       | AM memory *0.1, 最小值为  384                | 跟 *spark.driver.memoryOverhead* 类似，但是是用于 client 模式下的 YARN Application Master |
| spark.yarn.queue                                  | default                                      | 应用程序提交到 YARN 的队列名称                               |
| spark.yarn.jars                                   | none                                         | 包含 Spark 代码的库列表，会被分配到 YARN 的 containers 中。Spark on YARN 会使用本地的 jar 包，但是 Spark jar 包也可以在 HDFS 上。这个允许 YARN 在节点上缓存它，不需要每次应用程序运行时再分配它。要指定 HDFS 上的 jar 包，可以配置为 *hdfs:///some/path* |
| spark.yarn.archive                                | none                                         | 一个包含需要分配到 YARN 缓存的 Spark jar 包存档。如果设置了这个参数，它会替代 *spark.yarn.jars*, 并且存档会应用在所有应用程序的 container 中。存档应该在它的根目录中包含 jar 文件，存档也可以放在 HDFS 上来加快文件的分发 |
| spark.yarn.access.hadoopFileSystems               | none                                         |                                                              |
| spark.yarn.appMasterEnv.[EnvironmnetVariableName] | none                                         | 通过 *EnvironmentVariableName* 添加环境变量到 Application Master 进程中。用户可以添加多个环境变量。在 cluster 模式，这个可以控制 Spark driver 的环境变量，在 client 模式，它只控制 executor 的环境变量 |
| spark.yarn.containerLauncherMaxThread             | 25                                           | YARN Application Master 用来加载 executor container 的最大线程数 |
| spark.yarn.am.extraJavaOptions                    | none                                         |                                                              |
| spark.yarn.am.extraLibraryPath                    | none                                         |                                                              |
| spark.yarn.maxAppAttempts                         | yarn.resourcemanager.am.max-attempts in YARN | 提交应用程序的最大重试次数，这个值不能比 YARN 配置的最大次数大 |
| spark.yarn.am.attemptFailuresValidityInterval     | none                                         |                                                              |
| spark.yarn.executor.failuresValidityInterval      | none                                         |                                                              |
| spark.yarn.submit.waitAppCompletion               | true                                         | 在 YARN cluster 模式，控制client 是否等 application 完成后在退出。如果设置为 *true*，client 进程会一直存活并报道 application 的状态。否则，client 进程会在提交后退出 |
| spark.yarn.am.nodeLabelExpression                 | none                                         |                                                              |
| spark.yarn.executor.nodeLabelExpression           | none                                         |                                                              |
| spark.yarn.tags                                   | none                                         | 逗号分隔的字符串列表，作为 YARN 应用程序的标签出现在 YARN ApplicatioReports 中，可以通过查询 YARN 应用程序 |
| spark.yarn.keytab                                 | none                                         |                                                              |
| spark.yarn.principal                              | none                                         |                                                              |
| spark.yarn.kerberos.relogin.period                | 1m                                           |                                                              |
| spark.yarn.config.gatewayPath                     | none                                         |                                                              |
| spark.yarn.config.replacementPath                 | none                                         | 参照 *spark.yarn.config.geteeayPath*                         |
| spark.security.credentials.${service}.enabled     | true                                         |                                                              |
| spark.yarn.rolledLog.includePattern               | none                                         | Java Regex 用于过滤与定义的规则匹配的日志文件，这些文件会以滚动的方式进行聚合。这个会使用 YARN 滚动日志聚合，这YARN 中可以配置 yarn-site.xml 文件中的 *yarn.nodemanager.log-aggregation.roll-monitoring-interval-seconds* 配置项。这个特性只能使用在 hadoop 2.6.4 以上的版本中。Spark log4j 的 appender 需要配置为 FileAppender,或其它可以在运行是移除文件的配置。基于 log4j 中配置的文件名(比如 spark.log)，用后应该设置可以包含所有需要聚合的日志的匹配规则(比如 spark*) |
| spark.yarn.rolledLog.excludePattern               | none                                         | Java Regex 用来过滤跟排除规则匹配的日志文件，这些文件不会被聚合。如果日志文件名跟 exclude 和 include 都匹配，这个文件会被排除 |
|                                                   |                                              |                                                              |

### 重要提示

- 核心请求在调度决策中是否得到相应，取决于使用的调度程序和配置方式
- 在 cluster 模式，被 Spark executor 和 driver 使用的本地目录必须是通过 YARN 配置的目录(Hadoop YARN *yarn.nodemanager.local-dirs* 配置)。如果用户指定 *spark.local.dir* 则会被忽略。在client 模式，Spark executor 会使用 YARN 配置的本地目录，Spark deiver 会使用 *spark.local.dir* 配置的目录，这是因为在 client 模式 下 Spark driver 没有运行在 YARN 集群上，只有 Spark executor 运行在 YARN 集群上。
- *--files* 和 *--archives* 支持类似于 hadoop 中的 `#`，比如，你可以指定 *--files localtest.txt#appSees.txt* , 这个会上传本地名为 *localtest.txt* 的文件到 HDFS，但是连接到 *appSees.txt* ，如果应用程序运行在 YARN，需要指定 *appSees.txt* 来访问这个文件。
- 在 cluster 模式，如果通过 SparkContext.addJar 函数来访问本地文件，需要通过 *--jars* 先添加。如果访问的是 HDFS，HTTP，HTTPS 或 FTP 文件，则不需要通过 *--jars* 添加。

### 配置外部的 Shuffle 服务

在 YARN 集群上开启每个 NodeManager 的 Spark Shuffle 服务，可以通过下面的步骤：

1. 通过 YARN 构建 Spark ，如果使用的预安装包，可以跳过这步
2. 查找  *spark-\<version>-yarn-shuffle.jar* 的位置。如果是自己构建的 Spark , 应该是在 *$SPARK_HOME/common/network-yarn/target/scala-\<version>* 目录下, 如果使用的分布式，应该在 *yarn* 目录下
3. 添加这个 jar 包到集群中所有的 NodeManager 中
4. 在每个节点的 yarn-site.xml 文件中，添加 *spark_shuffle* 到 *yarn.nodemanager.aux-services*，然后设置 *yarn.nodemanager.aux-services.spark_shuffle.class* 为 *org.apache.spark.network.yarn.YarnShuffleService*
5. 通过设置 etc/hadoop/yarn-env.sh 文件中的 *YARN_HEAPSIZE* 来增加 NodeManager 的堆内存(默认 1000)，防止在 shuffle 期间的垃圾回收问题
6. 重启集群中所有的 NodeManager

当运行 shuffle 服务在 YARN 时，可以使用下面这个配置：

| Property Name                    | Default | Meaning                                                      |
| -------------------------------- | ------- | ------------------------------------------------------------ |
| spark.yarn.shuffle.stopOnFailure | false   | 在 Spark Shuffle 服务失败时是否停止 NodeManager。这个用来防止在 Spark Shuffle 服务没有的运行的 NodeManager 上运行应用程序造成的失败 |

### 使用 Spark History 服务替代 Spark Web UI

当 application UI 不能使用的时候，可以使用 Spark History 服务作为正在运行程序的 track URL，可以通过下面的步骤来设置 Spark History 服务：

- 在应用程序边，设置 Spark 的 *spark.yarn.historyServer.allowTracking=true* 配置，这个会告诉 Spark 在application 不能使用的时候，使用 history 服务的 URL 作为 tracking URL
- 在 Spark History 服务，添加 *org.apache.spark.deploy.yarn.YarnProxyRedirectFilter* 到 *spark.ui.filters* 配置

> history 服务不会更新应用程序的状态

# Spark SQL

## 简介

Spark SQL 是 Spark 用来处理结构化数据的模块。

### Datasets 和 DataFrames

Dataset 是一个分布式数据集，是在 Spark 1.6 的时候添加的，它具有 RDD 的有点：强类型并支持 lambda 函数，并且可以通过 Spark SQL 的优化引擎来执行计算。Dataset 支持 Scala 和 Java 语言，但是不支持 Python。

DataFrame 是给 Dataset 中的每列添加名字，它在概念上给表相近。DataFrame 支持 Scala, Java, Python 和 R。在 Scala 中 DataFrame 是 Dataset[Row] 的别名，在 Java 中，需要使用 Dataset<Row\> 代表 DataFrame。

 

