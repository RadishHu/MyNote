# Spark Streaming消费Kafka

> 参考：[Spark Streaming 消费kafka到HDFS](http://bigdatadecode.club/Spark%20Streaming%20%E6%B6%88%E8%B4%B9kafka%E5%88%B0HDFS.html)

Spark消费Kafka有两种方式

## Direct Approach(No Receivers)

Direct Approach将kafka数据源包裹成一个KafkaRDD，RDD里的partition对应的数据源为kafka的partition，有利于并行的读取kafka message

Kafka message并不是立马被读取spark内存，而是在kafka存着，直到有实际的Action被触发，才会去kafka主动拉数据

Direct Approach使用的是kafka simple consumer api，这样可以指定从某个offset处进行读取，有利于故障恢复

## Spark Streaming consumer Kafka to HDFS

这里主要讲message通过spark streaming根据不同的topic写到不同的hdfs文件中，并且能够记录消费message的offset，以支持故障恢复

offset存储方案选择:

- 利用checkpoint将offset存储在hdfs

  容易实现，根据需求有一定的局限，无法更好的满足需求

- 

