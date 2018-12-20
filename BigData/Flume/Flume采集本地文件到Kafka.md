# Flume采集本地文件到Kafka

- 配置文件

  cd Flume_Home/conf

  vi flume-dirKafka.properties

  ```properties
  #agent1 name
  agent1.sources=source1
  agent1.sinks=sink1
  agent1.channels=channel1
  
  #Spooling Directory
  #set source1
  agent1.sources.source1.type=spooldir
  agent1.sources.source1.spoolDir=/usr/app/flumelog/dir/logdfs
  agent1.sources.source1.channels=channel1
  agent1.sources.source1.fileHeader = false
  agent1.sources.source1.interceptors = i1
  agent1.sources.source1.interceptors.i1.type = timestamp
  
  #set sink1
  agent1.sinks.sink1.type = org.apache.flume.sink.kafka.KafkaSink
  agent1.sinks.sink1.topic = flumelog
  agent1.sinks.sink1.brokerList = cdh1:9092,cdh2:9092,cdh3:9092
  agent1.sinks.sink1.requiredAcks = 1
  agent1.sinks.sink1.batchSize = 100
  agent1.sinks.sink1.channel = channel1
  
  #set channel1
  agent1.channels.channel1.type=memory
  agent1.channels.channel1.capacity=1000
  agent1.channels.channel1.transactionCapacity=100
  ```

- 建立Linux目录

  ```
  mkdir /usr/app/flumelog/dir
  mkdir /usr/app/flumelog/dir/logdfs
  ```

- 建立Kafka的Topic

  ```
  bin/kafka-topics.sh --create --topic Flumelog --replication-factor 1 --partitions 2 --zookeeper hadoop11:2181
  ```

  > replication-factor：设置副本数，设为2，一共存储3份
  >
  > partitions：分区数

- 启动Flume的配置文件

  ```
  flume-ng agent -n agent1 -c conf -f ./flume-dirKakfa.properties
  -Dflume.root.logger=DEBUG,console >./flume1.log 2>&1 &
  ```

- 建立测试数据

- 启动Kafka消费者，分别在hadoop11和Hadoop12

> 日志监控路径，只要日志放入则被Flume监控
>
> 日志文件读取完毕，日志一直存储在Channel中，最多只有两个log-number日志，默认达到1.6G之后删除前面一个log，建立新的log