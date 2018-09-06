# Spark算子-键值转换操作

- partitionBy

  partitionBy函数根据partitioner函数生成新的ShuffleRDD，将原RDD重新分区

  def partitonBy(partitioner:Partitioner):RDD[(K,V)]

  ```scala
  scala> var rdd1 = sc.makeRDD(Array((1,"A"),(2,"B"),(3,"C"),(4,"D")),2)
  rdd1: org.apache.spark.rdd.RDD[(Int, String)] = ParallelCollectionRDD[23] at makeRDD at :21
   
  scala> rdd1.partitions.size
  res20: Int = 2
   
  //查看rdd1中每个分区的元素
  scala> rdd1.mapPartitionsWithIndex{
       |         (partIdx,iter) => {
       |           var part_map = scala.collection.mutable.Map[String,List[(Int,String)]]()
       |             while(iter.hasNext){
       |               var part_name = "part_" + partIdx;
       |               var elem = iter.next()
       |               if(part_map.contains(part_name)) {
       |                 var elems = part_map(part_name)
       |                 elems ::= elem
       |                 part_map(part_name) = elems
       |               } else {
       |                 part_map(part_name) = List[(Int,String)]{elem}
       |               }
       |             }
       |             part_map.iterator
       |            
       |         }
       |       }.collect
  res22: Array[(String, List[(Int, String)])] = Array((part_0,List((2,B), (1,A))), (part_1,List((4,D), (3,C))))
  //(2,B),(1,A)在part_0中，(4,D),(3,C)在part_1中
   
  //使用partitionBy重分区
  scala> var rdd2 = rdd1.partitionBy(new org.apache.spark.HashPartitioner(2))
  rdd2: org.apache.spark.rdd.RDD[(Int, String)] = ShuffledRDD[25] at partitionBy at :23
   
  scala> rdd2.partitions.size
  res23: Int = 2
   
  //查看rdd2中每个分区的元素
  scala> rdd2.mapPartitionsWithIndex{
       |         (partIdx,iter) => {
       |           var part_map = scala.collection.mutable.Map[String,List[(Int,String)]]()
       |             while(iter.hasNext){
       |               var part_name = "part_" + partIdx;
       |               var elem = iter.next()
       |               if(part_map.contains(part_name)) {
       |                 var elems = part_map(part_name)
       |                 elems ::= elem
       |                 part_map(part_name) = elems
       |               } else {
       |                 part_map(part_name) = List[(Int,String)]{elem}
       |               }
       |             }
       |             part_map.iterator
       |         }
       |       }.collect
  res24: Array[(String, List[(Int, String)])] = Array((part_0,List((4,D), (2,B))), (part_1,List((3,C), (1,A))))
  //(4,D),(2,B)在part_0中，(3,C),(1,A)在part_1中
  ```

- mapValues

  类似与基本转换操作中的map，只是mapValues是针对k-v对中的v值进行map操作

  def mapValues\[U](f:(V)=>U):RDD[K,U)]

  ```scala
  scala> var rdd1 = sc.makeRDD(Array((1,"A"),(2,"B"),(3,"C"),(4,"D")),2)
  rdd1: org.apache.spark.rdd.RDD[(Int, String)] = ParallelCollectionRDD[27] at makeRDD at :21
   
  scala> rdd1.mapValues(x => x + "_").collect
  res26: Array[(Int, String)] = Array((1,A_), (2,B_), (3,C_), (4,D_))
  ```

- flatMapValues

  类似与基本转换中的flatMap，flatMapValues是针对k-v对中的v值进行flatMap操作

- foldByKey

  该函数用于RDD[K,V]，根据K将V做合并处理