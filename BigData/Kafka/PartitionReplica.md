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

  