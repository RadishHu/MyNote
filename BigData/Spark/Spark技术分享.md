# 1."我从哪里来"

## 大数据处理技术发展过程

超大规模数据处理技术发展分为三个阶段：石器时代，青铜时代，蒸汽时代。

**石器时代**

石器时代表示 MapReduce 诞生之前的时代，在这个时期数据大规模处理问题已经存在，但是没有一个系统的处理方式(比如，做后台服务开发的 Spring，做前端开发的 Vue 等)，每个公司都有自己一套工具处理数据。

**青铜时代**

2003 年，MapReduce 的诞生标志着超大规模数据处理的第一次革命，而开创青铜时代的正式这篇论文《MapReduce: Simplified Data Processing on Large Clusters》。

![](https://static001.geekbang.org/resource/image/ae/61/ae9083e7b1f5cdd97deda1c8a1344861.png)

这篇论文的作者从负责的业务逻辑中，抽象出了 Map 和 Reduce 这种通用的编程模型。

**蒸汽机时代**

Spark、Storm、flink等计算框架的出现标志着青铜时代的终结和蒸汽机时代的开始。

## MapReduce

**归并排序的分治思想**

排序是计算机领域非常经典，也是非常流行的问题，归并排序和快速排序都很好的体现了分治思想。归并排序算法的核心就是“归并”，也就是把两个有序的数列合并起来，形成一个更大的有序数列。

假设需要按照从小到大的顺序，合并两个有序数列 A 和 B。需要开辟一个新的存储空间 C，用于保存合并 后的结果。首先比较两数列的第一个数，如果 A 数列的第一个数小于 B 数列的第一个数，那么就先取出 A 数列的第一个数放入 C，并把这个数从 A 数列删除。以此类推，直到 A 或 B 数列为空，那直接将另一个数列的数据一次放入 C 就可以了。这种操作就保证 C 数列仍然有序。

但是等待排序的数组一开始并不是有序的，那如何使用归并呢，可以利用递归的思想，把数列不断简化，一直简化到只剩 1 个数，1 个数本身就是有序的。因此，在归并排序中引入了分治思想。

归并排序通过分治思想，把长度为 n 的数列，每次简化为两个长度的 n/2 的数列，最后每个数列的长度为 1，需要进行 log<sub>2</sub>n 次拆分。把归并和分治思想结合起来，就是归并排序算法。

![](https://static001.geekbang.org/resource/image/54/12/5410fb301ffce57355ad7ef074e8fd12.jpg)

**分布式系统中的分治思想**

分布式系统应用的其实就是分治思想。当需要排序的数组很大，比如 1T(1024 GB) 的时候，没法把这些数据塞入一台普通机器的内存中。为了解决这个问题，可以把这个超大的数据集，分解为多个更小的数据集，然后分配到多台机器，让它们并行地处理。

等所有机器处理完之后，中央服务器再进行结果合并，多个小任务间不会相互干扰，可以同时处理，这样大大增加了处理的速度。

在单台机器上实现归并排序的时候，只需要在递归函数内实现数据分组以及合并，而在多台机器之间分配数据的时候，递归函数内除了分组及合并，还要负责把数据分发到某台机器上。

![](https://static001.geekbang.org/resource/image/78/31/78eefc6b61bad62f257f2b5e4972f031.jpg)

上述的分布式架构可以转换为类似的 MapReduce 的架构：

![](https://static001.geekbang.org/resource/image/08/5a/08155dd375f7b049424a6686bcb6475a.jpg)

这里主要由三个步骤用到了分治思想：

1. 数据分割和映射

   分割是指将数据源进行切分，并将分片发送到 Mapper 上。映射是指 Mapper 根据应用需求，将内容按照 K-V 匹配，存储到哈希结构中。

2. 规约

   规约是指接受到的一组 K-V 对，如果 Key 内容相同，就将它们的值归并。MapReduce 还需要 shuffle，也就是将 K-V 对不断发给对应的 Reducer 进行规约。

3. 合并

   为了提升 shuffle 的效率，可以选择减少发送到 Reduce 阶段的 K-V 对。具体做法实在数据映射和洗牌之间加入合并(combiner)，在么个 Mapper 节点先进行一次本地的归约

WordCoun.java

```java
public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

# 2."我是谁"

由于 MapReduce 面对日益复杂的业务逻辑时表现出的不足之处：1. 维护成本高；2. 时间性能不足。催生了下一代大规模数据处理技术。

为了解决 MapReduce 中复杂的 Map Reduce 关系，使用有向无环图 (DAG) 来抽象表达。有向无环图的经典示例：

![](https://static001.geekbang.org/resource/image/26/83/26072f95c409381f3330b77d93150183.png)

> 如果使用 MapReduce 来实现的话，每个箭头都会是一个独立的 Map 或 Reduce。

如果使用有向无环图建模，图中的每个节点都可以抽象地表达为一种通用的**数据集**，每个箭头都被表达为一种通用的**数据转换**。可以用数据集合数据变换描述极为宏大复杂的数据处理流程，而不会迷失在依赖关系中无法自拔。

## Spark

Spark 是当今最流行的分布式大规模数据处理引擎。

2009 年，美国加州大学伯克利分校的 AMP 实验室开发了 Spark。2013 年，Spark 成为 Apache 软件基金会旗下的孵化项目。

## RDD

Spark 最基本的数据抽象叫做弹性分布式数据集(Resilient Distributed Dataset, RDD)，它代表一个可以被分区 (partition) 、不可变、可以被并行操作的数据集。

**分区**

分区代表同一个 RDd 包含的数据被存储在系统的不同节点上，这是可以被并行处理的前提。

逻辑上，可以人为 RDD 是一个大的数组，数组中的每个元素代表一个分区 (Partition)。

在物理存储中，每个分区指向一个存放在内存或磁盘中的数据块 (Block)，而这些数据块是独立的，可以被存放在系统中的不同节点。

因此，RDD 只是抽象意义的数据集合，分区内部并不会存储具体的数据，下图展示了 RDD 的分区逻辑结构：

![](https://static001.geekbang.org/resource/image/2f/9e/2f9ec57cdedf65be382a8ec09826029e.jpg)

RDD 中的每个分区存有它在该 RDD 中的 index。通过 RDD 的 ID 和分区的 index 可以唯一确定对应数据块的编号，从而通过底层存储层的接口提取数据进行处理。

**不可变性**

不可变性代表每一个 RDD 都是只读的，它所包含的分区信息不可以改变，只可以对现有的 RDD 进行转换操作，得到新的 RDD。

**并行操作**

由于单个 RDD 的分区特性，使得它天然支持并行操作，即不同节点上的数据可以被分别处理，然后生成一个新的 RDD。

实际上 RDD 的结构要比想象的复杂，RDD 的简易结构示意图：

![](https://static001.geekbang.org/resource/image/8c/1c/8cae25f4d16a34be77fd3e84133d6a1c.png)

SparkContext 是所有 Spark 功能的入口，它代表与 Spark 节点的连接。可以用来创建 RDD 对象以及在节点中的广播变量，一个线程只有一个 SparkContext。SparkConf 是一些参数的配置信息。

Partitions 代表 RDD 中数据的逻辑结构，每个 Partition 会映射到某个节点内存活硬盘的一个数据块。

Partitioner 决定了 RDd 的分区方式，目前有两种主流的分区方式：Hash partitioner 和 Range partitions。Hash 是对数据的 Key 进行散列分区，Range 是按照 Key 的排序进行均匀分区。此外也可以创建自定义的 Partitioner。

**依赖关系** (Dependencies)

Dependencies 是 RDD 中最重要的组件之一，Spark 不需要将每个中间计算结果进行数据复制以防止数据丢失，因为每一步产生的 RDD 都会存储它的依赖关系，即他是通过哪个 RDD 经过哪个转换操作得到的。

Spark 支持两种依赖关系：窄依赖 (Narrow Dependency) 和宽依赖 (Wide Dependency)。

![](https://static001.geekbang.org/resource/image/5e/e1/5eed459f5f1960e2526484dc014ed5e1.jpg)

窄依赖就是父 RDD 的分区可以一一对应到子 RDD 的分区，宽依赖就是父 RDD 的每个分区可以被多个子 RDD 的分区使用。

![](https://static001.geekbang.org/resource/image/98/f9/989682681b344d31c61b02368ca227f9.jpg)

窄依赖允许子 RDD 的每个分区可以被并行处理产生，而宽依赖则必须等父 RDD 的所有分区被计算好之后才能开始处理。

Spark 区分宽依赖和窄依赖出于两点考虑：

- 窄依赖可以支持在同一个节点上链式执行多条命令，例如在执行 map 后，紧接着执行 filter。宽依赖需要所有的父分区都是可用的，可能还需要进行跨节点传输。
- 从失败恢复的角度考虑，窄依赖的恢复更高效，因为它只需要重新计算丢失的分区即可，而宽依赖牵涉到 RDD 各级的多个父分区。



# 3."将到哪里去"





# SparkSession

在 Spark 早起版本，SparkContext 作为 Spark 的切入点，可以通过 SparkContext 创建 RDD 和操作 RDD。但是对于 RDD 之外的东西，需要使用其他的 Context，比如，对于流处理需要使用 StreamingContext，对于 SQL 需要使用 SqlContext，对于 hive 需要使用 HiveContext。DataSet 和 DataFrame 的 API 主键成为新的标准，需要一个新的切入点来构架它们，所以在 Spark 2.0 中引入了新的切入点 SparkSession。

SparkSession 实质上是 SQLContext 和 HiveContext 的组合(还没有加入 StreamingContext)，所以在 SQLContext 和 Hive Context 中可以使用的 API 在 SparkSession 中都可以使用。SparkSession 内封装了 SparkContext，所以计算实际是由 SparkContext 完成的。

# 参考

[Spark 2.0介绍：SparkSession创建和使用相关API](https://www.iteblog.com/archives/1673.html)