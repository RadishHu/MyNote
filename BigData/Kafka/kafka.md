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



