# Kafka

Kafka 是一个消息队列(MQ)，消息队列是存放消息的容器

# Kafka 的作用

- 削峰

- 解耦

# Kafka 架构原理

## Kafka 相关概念

- Break: Kafka 集群中的一台服务器就是一个 Break
- Producer: 消息生产者
- Consumer: 消息消费者
- Consumer Group: 消费组。每个 Consumer 都属于一个 Consumer Group，每条消息只能被 Consumer Group 中的一个 Consumer 消费，但是可以被多个 Consumer Group 消费
- Topic: 消息的类型。每条消息都属于某个 Topic，Kafka 是面向 Topic 的
- Partition: 每个 Topic 分为多个 Partition，Partition 是 kafka 分配单位。Partition 在 Kafka 中的物理上的概念，相当于一个目录，目录下的日志文件构成 Partition
- Replica: Partition 的副本，保证 Partition 的高可用
- Leader: Replica 中的一个角色，Producer 和 Consumer 只跟 Leader 交互
- Follower: Replica 中的一个角色，从 Leader 中复制数据
- Controller: Kafka 集群中的一个服务器，用来进行 Leader Election 以及各种 Failover
- Zookeeper: Kafka 通过 Zookeeper 来存储集群中的 Meta 信息

## Topic & Logs

Message 是按照 Topic 来组织的，每个  Topic 可以分成多个 Partiton(Partition 的数量由 service.properties/num.partitions 控制)

Partition 是一个顺序的追加日志，属于顺序写磁盘(顺序写磁盘效率比随机写内存高，保障了 Kafka 的吞吐率)

Partition 中的每条记录(Message) 包含三个属性: 

- Offset，表示消息的偏移量，Long 类型(64位)
- MessageSize，表示消息的大小，int 类型(32位)
- Data，表示消息的具体内容

Partition 以文件的形式存储在本地文件系统中，位置由 server.properties/log.dirs 指定，其命名规则为 <topic_name>-<partition_id>

Partition 是分段的，每个段是一个 Segment 文件。Segment 常用的配置有: 

```properties
# server.properties

# segment 文件的大小，默认为 1G
log.segment.bytes=1024*1024*1024
# 滚动生成新的 segment 文件的最大时长
log.roll.hour=24*7
# segment 文件保留的最大时间，超过时间会被删除
log.retention.hour=24*7
```

segment 文件命名规则，partition 的第一个 segment 从 0 开始，后续每个 segment 文件命名为上一个 segment 文件最后一条消息的 offset。segment 文件包括 index file 和 data file

Partition 目录下包括 log (数据)文件和 index (索引)文件。

index 文件采用稀疏存储的方式，它不会为每一条 Message 都建立索引，而是每隔一定的字节数建立一条索引，避免索引文件占用过多的空间。采用稀疏索引不能一次定位到 Message 的位置，需要做一个顺序扫描，但是扫描的范围很小。

idnex 包含两部分(均为 4 个字节的数字)，分别为:

- 相对 Offset，表示 Message 在 Segment 文件中的 Offset
- Position，表示 Message 自带的 Offset

Kakfa 采用 Partition(分区), 磁盘顺序读写, LogSegment(分段)和稀疏索引这几个方法来达到高效性。LogSegment 和稀疏索引保证根据索引快速定位数据，Partition 和磁盘顺序读写保证 Kafka 高吞吐量。

## Partition & Replica

一个 Topic 物理上分为多个 Partition，位于不同的 Broker 上。如果没有 Replica，一旦 Broker 宕机，其上所有的 Partition 将不可用

每个 Partition 可以有多个 Replica(通过 server.properties/defaut.replication.factor 设置)，分配到不同 Broker 上。

多个 Replica 分为一个 Leader 和多个 Follower，Leader 负责读写，处理来自 Producer 和 Consumer 的请求；Follower 从 Leader Pull 消息，保持与 Leader 的消息同步。

分配 Partition 和 Replica 到不同的 Broker: 

- 将所有 Broker (n 个) 和待分配的 Partition 排序
- 将第 i 个 Partition 分配到第 (i mod n) 个 Broker 上
- 将第 i 个 Partition 的第 j 个 Replica 分配到第 ((i + j) mod n) 个 Broker 上

> 根据这个分配规则，如果 Replica 的数量大于 Broker 的数量，就会有两个相同的 Replica 分配到同一个 Broker 上，产生冗余，因此 Replica 的数量应该小于或等于 Broker 的数量

## Controller

Controller 是 Kafka Server 端的一个重要组件，它的角色类似于其他分布式系统 Master 的角色，不同之处在于 Kafka 集群中的任何一台 Broker 都可以作为 Controller。

Controller 的作用包括：

- 保证集群 meta 信息一致
- Partition leader 选举
- Broker 上下线

## Zookeeper

Kafka 集群的一部分信息维护在 ZK 中：

- /controller，记录哪个 Broker 作为 Controller

  ```
  get /controller
  
  {"version":1,"brokerid":0,"timestamp":"1555399391766"}
  ```

- /brokers/ids，记录 Kafka 集群的所有 Broker id

  ```
  ls /brokers/ids
  
  [0, 1, 2]
  
  get /brokers/ids/0
  
  {"listener_security_protocol_map":{"PLAINTEXT":"PLAINTEXT"},"endpoints":["PLAINTEXT://cdh1:9092"],"jmx_port":9
  393,"host":"node1","timestamp":"1555399394838","port":9092,"version":4}
  ```

  

