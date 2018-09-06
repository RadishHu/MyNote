# RDD基本转换操作

- map

  将一个RDD中的每个数据项，通过map中的函数映射变为一个新的元素

  输入分区与输出分区一对一，即输入多少个分区，就有多少个输出分区

  ```scala
  hadoop fs -cat /tmp/1.txt
  hello world
  hello spark
  hello hive
  
  //读取HDFS文件到RDD
  scala> var data = sc.textFile("/tmp/lxw1234/1.txt")
  data: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[1] at textFile at :21
   
  //使用map算子
  scala> var mapresult = data.map(line => line.split("\\s+"))
  mapresult: org.apache.spark.rdd.RDD[Array[String]] = MapPartitionsRDD[2] at map at :23
   
  //运算map算子结果
  scala> mapresult.collect
  res0: Array[Array[String]] = Array(Array(hello, world), Array(hello, spark), Array(hello, hive))
  ```

- flatMap

  第一步和map一样，最后将所有的输出分区合并成一个

  ```scala
  //使用flatMap算子
  scala> var flatmapresult = data.flatMap(line => line.split("\\s+"))
  flatmapresult: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at flatMap at :23
   
  //运算flagMap算子结果
  scala> flatmapresult.collect
  res1: Array[String] = Array(hello, world, hello, spark, hello, hive)
  ```

  > flatMap会将字符串看成是一个字符数组：
  >
  > ```scala
  > scala> data.map(_.toUpperCase).collect
  > res32: Array[String] = Array(HELLO WORLD, HELLO SPARK, HELLO HIVE, HI SPARK)
  > scala> data.flatMap(_.toUpperCase).collect
  > res33: Array[Char] = Array(H, E, L, L, O,  , W, O, R, L, D, H, E, L, L, O,  , S, P, A, R, K, H, E, L, L, O,  , H, I, V, E, H, I,  , S, P, A, R, K)
  > ```

- mapPartitions

  mapPartitions和map函数类似，只不过映射函数的参数由RDD中每一个元素变为RDD中每个分区的迭代器。如果在映射过程中需要频繁创建额外的对象，使用mapPartition要比map高效

  def mapPartitions[U](f:(Iterator[T] => Iterator[U],preservesPartitioning:Boolean=false)(implicit arg0:ClassTag[U]):RDD[U]

  > 参数preservesPartitioning表示是否保留父RDD的partitioner分区信息

  ```scala
  var rdd1 = sc.makeRDD(1 to 5,2)
  //rdd1有两个分区
  scala> var rdd3 = rdd1.mapPartitions{ x => {
       | var result = List[Int]()
       |     var i = 0
       |     while(x.hasNext){
       |       i += x.next()
       |     }
       |     result.::(i).iterator
       | }}
  rdd3: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[84] at mapPartitions at :23
   
  //rdd3将rdd1中每个分区中的数值累加
  scala> rdd3.collect
  res65: Array[Int] = Array(3, 12)
  scala> rdd3.partitions.size
  res66: Int = 2
  ```

- distinct

  对RDD中的元素进行去重操作

  ```scala
  scala> data.flatMap(line => line.split("\\s+")).collect
  res61: Array[String] = Array(hello, world, hello, spark, hello, hive, hi, spark)
   
  scala> data.flatMap(line => line.split("\\s+")).distinct.collect
  res62: Array[String] = Array(hive, hello, world, spark, hi)
  ```

- coalesce

  coalesce函数用于将RDD进行重新分区，使用HashPartitioner

  def coalesce(numPartitions:Int, shuffle:Boolean = false)

  > * 第一个参数为重分区的数目
  > * 第二个为是否进行shuffle，默认为false

  ```scala
  scala> var data = sc.textFile("/tmp/lxw1234/1.txt")
  data: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[53] at textFile at :21
   
  scala> data.collect
  res37: Array[String] = Array(hello world, hello spark, hello hive, hi spark)
   
  scala> data.partitions.size
  res38: Int = 2  //RDD data默认有两个分区
   
  scala> var rdd1 = data.coalesce(1)
  rdd1: org.apache.spark.rdd.RDD[String] = CoalescedRDD[2] at coalesce at :23
   
  scala> rdd1.partitions.size
  res1: Int = 1   //rdd1的分区数为1
   
  scala> var rdd1 = data.coalesce(4)
  rdd1: org.apache.spark.rdd.RDD[String] = CoalescedRDD[3] at coalesce at :23
   
  scala> rdd1.partitions.size
  res2: Int = 2   //如果重分区的数目大于原来的分区数，那么必须指定shuffle参数为true，//否则，分区数不变
   
  scala> var rdd1 = data.coalesce(4,true)
  rdd1: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[7] at coalesce at :23
   
  scala> rdd1.partitions.size
  res3: Int = 4
  ```

- repartition

  repartition函数是coalesce函数第二个参数为true的实现：

  def repartition(numPartitions:Int)

  ```scala
  scala> var rdd2 = data.repartition(1)
  rdd2: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[11] at repartition at :23
   
  scala> rdd2.partitions.size
  res4: Int = 1
   
  scala> var rdd2 = data.repartition(4)
  rdd2: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[15] at repartition at :23
   
  scala> rdd2.partitions.size
  res5: Int = 4
  ```

- randomSplit

  randomSplit函数根据weights权重，将一个RDD切分成多个RDD

  def randomSplit(weights:Array[Double],seed:Long = Utils.random.nextLong):Array[RDD[T]]

  > * 第一个参数weights，切分RDD的权重
  > * 第二个参数为random的种子，可以忽略

  ```scala
  scala> var rdd = sc.makeRDD(1 to 10,10)
  rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[16] at makeRDD at :21
   
  scala> rdd.collect
  res6: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)  
   
  scala> var splitRDD = rdd.randomSplit(Array(0.1,0.2,0.3,0.4))
  splitRDD: Array[org.apache.spark.rdd.RDD[Int]] = Array(MapPartitionsRDD[17] at randomSplit at :23, 
  MapPartitionsRDD[18] at randomSplit at :23, 
  MapPartitionsRDD[19] at randomSplit at :23, 
  MapPartitionsRDD[20] at randomSplit at :23)
   
  //这里注意：randomSplit的结果是一个RDD数组
  scala> splitRDD.size
  res8: Int = 4
  //由于randomSplit的第一个参数weights中传入的值有4个，因此，就会切分成4个RDD,
  //把原来的rdd按照权重1.0,2.0,3.0,4.0，随机划分到这4个RDD中，权重高的RDD，划分到//的几率就大一些。
  //注意，权重的总和加起来为1，否则会不正常
   
  scala> splitRDD(0).collect
  res10: Array[Int] = Array(1, 4)
   
  scala> splitRDD(1).collect
  res11: Array[Int] = Array(3)                                                    
   
  scala> splitRDD(2).collect
  res12: Array[Int] = Array(5, 9)
   
  scala> splitRDD(3).collect
  res13: Array[Int] = Array(2, 6, 7, 8, 10)
  ```

- glom

  glom函数将RDD中每一个分区中的元素转换成Array[T]，这样每个分区中就只有一个数组元素：

  def glom():RDD[Array[T]]

  ```scala
  scala> var rdd = sc.makeRDD(1 to 10,3)
  rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[38] at makeRDD at :21
  scala> rdd.partitions.size
  res33: Int = 3  //该RDD有3个分区
  scala> rdd.glom().collect
  res35: Array[Array[Int]] = Array(Array(1, 2, 3), Array(4, 5, 6), Array(7, 8, 9, 10))
  //glom将每个分区中的元素放到一个数组中，这样，结果就变成了3个数组
  ```

- union

  union函数将两个RDD进行合并，不去重

  ```scala
  scala> var rdd1 = sc.makeRDD(1 to 2,1)
  rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[45] at makeRDD at :21
   
  scala> rdd1.collect
  res42: Array[Int] = Array(1, 2)
   
  scala> var rdd2 = sc.makeRDD(2 to 3,1)
  rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[46] at makeRDD at :21
   
  scala> rdd2.collect
  res43: Array[Int] = Array(2, 3)
   
  scala> rdd1.union(rdd2).collect
  res44: Array[Int] = Array(1, 2, 2, 3)
  ```

- intersection

  intersection函数返回两个RDD的交集，并且去重

  def intersection(other:RDD[T]):RDD[T]

  def intersection(other:RDD[T],numPartitons:Int):RDD[T]

  def intersection(other:RDD[T],partitioner:Partitioner)(implicit ord:Ordering[T] = null):RDD[T]

  > * 参数numPartitions，指定返回RDD的分区数
  > * 参数partitioner，用于指定分区函数

  ```scala
  scala> var rdd1 = sc.makeRDD(1 to 2,1)
  rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[45] at makeRDD at :21
   
  scala> rdd1.collect
  res42: Array[Int] = Array(1, 2)
   
  scala> var rdd2 = sc.makeRDD(2 to 3,1)
  rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[46] at makeRDD at :21
   
  scala> rdd2.collect
  res43: Array[Int] = Array(2, 3)
   
  scala> rdd1.intersection(rdd2).collect
  res45: Array[Int] = Array(2)
   
  scala> var rdd3 = rdd1.intersection(rdd2)
  rdd3: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[59] at intersection at :25
   
  scala> rdd3.partitions.size
  res46: Int = 1
   
  scala> var rdd3 = rdd1.intersection(rdd2,2)
  rdd3: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[65] at intersection at :25
   
  scala> rdd3.partitions.size
  res47: Int = 2
  ```

- substract

  substract函数返回再RDD中出现，并且不在otherRDD中出现的元素，不去重

  def substract(other:RDD[T]):RDD[T]

  def substract(other:RDD[T],numPartitions:Int):RDD[T]

  def substract(other:RDD[T],partitioner:Partitioner)(implicit ord:Ordering[T] = null):RDD[T]

  ```scala
  scala> var rdd1 = sc.makeRDD(Seq(1,2,2,3))
  rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[66] at makeRDD at :21
   
  scala> rdd1.collect
  res48: Array[Int] = Array(1, 2, 2, 3)
   
  scala> var rdd2 = sc.makeRDD(3 to 4)
  rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[67] at makeRDD at :21
   
  scala> rdd2.collect
  res49: Array[Int] = Array(3, 4)
   
  scala> rdd1.subtract(rdd2).collect
  res50: Array[Int] = Array(1, 2, 2)
  ```

- zip

  zip函数用于将两个RDD组合成k-v形式的RDD，这里默认两个RDD的partition数以及元素数量都相同，否则会抛出异常

  ```scala
  scala> var rdd1 = sc.makeRDD(1 to 10,2)
  rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at makeRDD at :21
   
  scala> var rdd1 = sc.makeRDD(1 to 5,2)
  rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[1] at makeRDD at :21
   
  scala> var rdd2 = sc.makeRDD(Seq("A","B","C","D","E"),2)
  rdd2: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[2] at makeRDD at :21
   
  scala> rdd1.zip(rdd2).collect
  res0: Array[(Int, String)] = Array((1,A), (2,B), (3,C), (4,D), (5,E))           
   
  scala> rdd2.zip(rdd1).collect
  res1: Array[(String, Int)] = Array((A,1), (B,2), (C,3), (D,4), (E,5))
   
  scala> var rdd3 = sc.makeRDD(Seq("A","B","C","D","E"),3)
  rdd3: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[5] at makeRDD at :21
   
  scala> rdd1.zip(rdd3).collect
  java.lang.IllegalArgumentException: Can't zip RDDs with unequal numbers of partitions
  //如果两个RDD分区数不同，则抛出异常
  ```

- zipPartitions

  zipPartitions函数将多个RDD按照partiton组合成为新的RDD，该函数需要组合具有相同的分区数，但对于每个分区内的元素数量没有要求

  该函数有三种实现：

  - 参数是一个RDD

    def zipPartitions\[B, V](rdd2: RDD[B])(f: (Iterator[T], Iterator[B]) => Iterator[V])(implicit arg0: ClassTag[B], arg1: ClassTag[V]): RDD[V]

    def zipPartitions\[B, V](rdd2: RDD[B], preservesPartitioning: Boolean)(f: (Iterator[T], Iterator[B]) => Iterator[V])(implicit arg0: ClassTag[B], arg1: ClassTag[V]): RDD[V]

    这两个的区别在于参数preservesPartitioning，是否保留父RDD的partitioner分区信息

    映射方法f参数为两个RDD的迭代器

    ```scala
    scala> var rdd1 = sc.makeRDD(1 to 5,2)
    rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[22] at makeRDD at :21
     
    scala> var rdd2 = sc.makeRDD(Seq("A","B","C","D","E"),2)
    rdd2: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[23] at makeRDD at :21
     
    //rdd1两个分区中元素分布：
    scala> rdd1.mapPartitionsWithIndex{
         |         (x,iter) => {
         |           var result = List[String]()
         |             while(iter.hasNext){
         |               result ::= ("part_" + x + "|" + iter.next())
         |             }
         |             result.iterator
         |            
         |         }
         |       }.collect
    res17: Array[String] = Array(part_0|2, part_0|1, part_1|5, part_1|4, part_1|3)
     
    //rdd2两个分区中元素分布
    scala> rdd2.mapPartitionsWithIndex{
         |         (x,iter) => {
         |           var result = List[String]()
         |             while(iter.hasNext){
         |               result ::= ("part_" + x + "|" + iter.next())
         |             }
         |             result.iterator
         |            
         |         }
         |       }.collect
    res18: Array[String] = Array(part_0|B, part_0|A, part_1|E, part_1|D, part_1|C)
     
    //rdd1和rdd2做zipPartition
    scala> rdd1.zipPartitions(rdd2){
         |       (rdd1Iter,rdd2Iter) => {
         |         var result = List[String]()
         |         while(rdd1Iter.hasNext && rdd2Iter.hasNext) {
         |           result::=(rdd1Iter.next() + "_" + rdd2Iter.next())
         |         }
         |         result.iterator
         |       }
         |     }.collect
    res19: Array[String] = Array(2_B, 1_A, 5_E, 4_D, 3_C)
    ```

  - 参数是两个RDD

    def zipPartitions\[B, C, V](rdd2: RDD[B], rdd3: RDD[C])(f: (Iterator[T], Iterator[B], Iterator[C]) => Iterator[V])(implicit arg0: ClassTag[B], arg1: ClassTag[C], arg2: ClassTag[V]): RDD[V]

    def zipPartitions\[B, C, V](rdd2: RDD[B], rdd3: RDD[C], preservesPartitioning: Boolean)(f: (Iterator[T], Iterator[B], Iterator[C]) => Iterator[V])(implicit arg0: ClassTag[B], arg1: ClassTag[C], arg2: ClassTag[V]): RDD[V]

    用法同上面相同，只是该函数参数为两个RDD，映射方法f输入参数为两个RDD的迭代器

    ```scala
    scala> var rdd1 = sc.makeRDD(1 to 5,2)
    rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[27] at makeRDD at :21
     
    scala> var rdd2 = sc.makeRDD(Seq("A","B","C","D","E"),2)
    rdd2: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[28] at makeRDD at :21
     
    scala> var rdd3 = sc.makeRDD(Seq("a","b","c","d","e"),2)
    rdd3: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[29] at makeRDD at :21
     
    //rdd3中个分区元素分布
    scala> rdd3.mapPartitionsWithIndex{
         |         (x,iter) => {
         |           var result = List[String]()
         |             while(iter.hasNext){
         |               result ::= ("part_" + x + "|" + iter.next())
         |             }
         |             result.iterator
         |            
         |         }
         |       }.collect
    res21: Array[String] = Array(part_0|b, part_0|a, part_1|e, part_1|d, part_1|c)
     
    //三个RDD做zipPartitions
    scala> var rdd4 = rdd1.zipPartitions(rdd2,rdd3){
         |       (rdd1Iter,rdd2Iter,rdd3Iter) => {
         |         var result = List[String]()
         |         while(rdd1Iter.hasNext && rdd2Iter.hasNext && rdd3Iter.hasNext) {
         |           result::=(rdd1Iter.next() + "_" + rdd2Iter.next() + "_" + rdd3Iter.next())
         |         }
         |         result.iterator
         |       }
         |     }
    rdd4: org.apache.spark.rdd.RDD[String] = ZippedPartitionsRDD3[33] at zipPartitions at :27
     
    scala> rdd4.collect
    res23: Array[String] = Array(2_B_b, 1_A_a, 5_E_e, 4_D_d, 3_C_c)
    ```

  - 参数为三个RDD

    def zipPartitions\[B, C, D, V](rdd2: RDD[B], rdd3: RDD[C], rdd4: RDD[D])(f: (Iterator[T], Iterator[B], Iterator[C], Iterator[D]) => Iterator[V])(implicit arg0: ClassTag[B], arg1: ClassTag[C], arg2: ClassTag[D], arg3: ClassTag[V]): RDD[V]

    def zipPartitions\[B, C, D, V](rdd2: RDD[B], rdd3: RDD[C], rdd4: RDD[D], preservesPartitioning: Boolean)(f: (Iterator[T], Iterator[B], Iterator[C], Iterator[D]) => Iterator[V])(implicit arg0: ClassTag[B], arg1: ClassTag[C], arg2: ClassTag[D], arg3: ClassTag[V]): RDD[V]

- zipWithIndex

  zipWithIndex函数将RDD中的元素和这个元素在RDD中的ID(索引号)组成k-v对

  ```scala
  scala> var rdd2 = sc.makeRDD(Seq("A","B","R","D","F"),2)
  rdd2: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[34] at makeRDD at :21
   
  scala> rdd2.zipWithIndex().collect
  res27: Array[(String, Long)] = Array((A,0), (B,1), (R,2), (D,3), (F,4))
  ```

- zipWithUniqueId

  该函数将RDD中元素和一个唯一ID组合成k-v对，该唯一ID生成算法如下：

  每个分区中第一个元素的唯一ID值为：该分区索引号，

  每个分区中第N个元素的唯一ID值为：(前一个元素的唯一ID值) + (该RDD总的分区数)

  ```scala
  scala> var rdd1 = sc.makeRDD(Seq("A","B","C","D","E","F"),2)
  rdd1: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[44] at makeRDD at :21
  //rdd1有两个分区，
  scala> rdd1.zipWithUniqueId().collect
  res32: Array[(String, Long)] = Array((A,0), (B,2), (C,4), (D,1), (E,3), (F,5))
  //总分区数为2
  //第一个分区第一个元素ID为0，第二个分区第一个元素ID为1
  //第一个分区第二个元素ID为0+2=2，第一个分区第三个元素ID为2+2=4
  //第二个分区第二个元素ID为1+2=3，第二个分区第三个元素ID为3+2=5
  ```

  