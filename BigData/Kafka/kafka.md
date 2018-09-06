# Kafka简介

Kafka是一个分布式消息队列，对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，Producer推送数据，Consumer拉取数据

Kafka集群由多个实例组成，每个实例(server)成为broker

Kafka集群依赖于zookeeper保存meta信息

## Kafka基础概念

- Topic：一类消息，每个topic被分为多个partition，分区数可以在集群配置文件中配置
- Partition
  - 在存储层面时逻辑append log文件，每个partition由多个segment组成
  - 任何发布到此partition的消息都会被直接追加到log文件尾部
  - 每个partiton在内存中对应一个index列表，记录每个segment第一条消息偏移，查找消息时，先在index列表中定位，再读取文件，速度快
  - producer发布到某个topic的消息会被均匀发布到part上，broker受到发布消息往part的最后一个segment上添加消息
- Segment
  - 当segment上的消息条数达到配置值或消息发布时间超过阀值，segment上的消息会被flush到磁盘，只有flush到磁盘的消息，订阅者才能订阅到数据
  - segment达到一定的大小(默认1G)，不会再往该segment写数据，borker会创建新的segment
- offset
  - segment日志文件中保存了一系列log entries，log entries的格式为：消息长度 + 消息内容
  - segment日志文件都有一个offset来唯一标记一条消息，segment文件命名为：‘最小offset’.log

## Kafka保证数据不丢失

- Producer保证数据不丢失

  通过ack机制：再kafka发送数据的时候，每次发送消息都会有一个确认反馈机制，确保消息能被收到

- broker保证数据不丢失

  数据备份，副本机制

- Consumer保证数据不丢失

  offset，记录每次消费到哪一条数据，下次接着上次的消费记录进行消费