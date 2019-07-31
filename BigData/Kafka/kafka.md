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

