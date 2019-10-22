# Elasticsearch 基本概念

## 集群 (Cluster)

Elasticsearch 通过分布式集群可以实现高可用性和扩展性：

- 高可用性
  - 服务可用性，允许节点停止服务
  - 数据可用性，部分节点丢失，不会丢失数据
- 可扩展性
  - 请求 / 数据不断增加，将数据分散到所有的节点上

一个 Elasticsearch 集群可以有一个或多个节点，不同的集群通过不同的名字来区分，设定集群的名字的方法：

- 配置文件，通过 `conf/elasticsearch.yml` 配置文件的 `cluster.name` 配置项
- 启动命令，在启动命令中添加 `-E cluster.name=yourName` 

Elasticsearch 集群的健康状况通过颜色来表示：

- Green，主分片和副本分片都有正常分配
- Yellow，主分片全部正常分配，有副本分片未能正常分配
- Red，有主分片未分配

## Node

Node 是一个 Elasticsearch 实例，本质上是一个 Java 进程，一台机器可以运行多个 Node，生产环境建议一台机器上只运行一个 Node。每一个节点都有名字，节点名字设定的方法：

- 配置文件，通过 `conf/elasticsearch.yml` 配置文件的 `node.name` 配置项
- 启动命令，在启动命令中添加 `-E nod.name=yourName` 

每一个节点启动后，都会分配一个 UID，保存在 data 目录下。

Elasticsearch 的节点可以分为多种类型：

- Master Node

  集群中的每个节点都保存了集群的状态，但是只有 Master 节点才能修改集群的状态信息。集群的状态信息包括：

  - 所有的节点信息
  - 所有的索引和其相关的 Mapping 和 Setting 信息
  - 分片的路由信息

  每个节点启动后，默认都是一个 Master eligible (有资格当选 Master 的节点)，可以通过 配置文件的 `node.master: false` 来禁用，Master-eligible 节点可以参加选主流程，成为 Master 节点。

- Data Node

  负责保存分片数据

- Coordinating Node

  负责接受客户端的请求，将请求分发到合适的节点，最终把结果汇总到一起。每个节点默认都是 Coordinating (协调) 节点。

在开发环境中一个节点可以承担多种角色，在生产环境中，应该设置单一角色的节点。通过修改配置文件中对应参数来设定节点角色：

| 节点类型          | 配置参数    | 默认值                                                       |
| ----------------- | ----------- | ------------------------------------------------------------ |
| master            | node.master | true                                                         |
| data              | node.data   | true                                                         |
| coordinating only | 无          | 要设置 coordinating 单一角色的节点，可以设置其它类型参数为 false |

## 分片 (Shard)

分片分为两类：

- 主分片 (Primary Shard)，可以解决数据水平扩展的问题，通过主分片，可以将数据分布到集群内的所有节点上
  - 一个主分片是一个运行的 Lucene 实例
  - 主分片数在创建索引时指定，后续不允许修改，除非 Reindex
- 副本分片 (Replica Shard)，用于解决数据高可用问题，副本分片是主分片的拷贝
  - 副本分片数可以动态调整
  - 增加副本分片数，可以在一定程度上提高服务的可用性 (读取的吞吐)

在设定分片数时，需要提前做好容量规划：

- 分片数过小
  - 导致后续无法通过增加节点来实现水平扩展
  - 单个分片的数据量过大，导致数据重新分配耗时
- 分片数过大
  - 影响搜索结果的相关性打分，影响统计结果的准确性
  - 单个节点上过多的分片，会导致资源浪费，同时影响性能

## 文档 (Document)

文档是可以被索引的信息的最小单位，相当传统数据库中的一条数据。

- 文档会被序列化为 Json 格式进行保存，Json 对象由字段组成，每个字段都有对应的字段类型，字段类型可以在创建 Index 时指定，也可以通过 elasticsearch 自动推算类型
- 每个文档都有一个 UUID
  - 可以在写入文档时自己指定 ID
  - 也可以由 elasticsearch 自动生成

### 文档的元数据

元数据用于标注文档的相关信息：

- _index，文档的所属的索引名
- _type，文档所属的类型名
- _id，文档唯一 id
- _source，文档原始 Json 数据
- _all，整合所有字段内容到该字段，从 7.0 版本开始已经废除
- _version，文档的版本信息
- _score，相关性打分

元数据示例：

```json
{
    "_index":"movies",
    "_type":"_doc",
    "_id":"1",
    "_score":14.69302,
    "_source":{
        "year":1995,
    	"version":"1",
    	"genre":[
        	"Adventure","Animation",
        	"Children","Comedy","Fantasy"
    	],
    	"id":"1",
    	"title":"Toy Story"
    }
}
```

## 索引 (Index)

索引是文档的容器，如果不考虑 Type 的存在 (在 7.0 中已经废除)，索引类似于数据库中的表

- 索引是逻辑空间上的概念
- 分片是物理存储空间上的概念，索引中的数据分散在 Shard 上

索引的 Mapping 和 Settings:

- Mapping 用于定义所包含的文档的字段名和字段类型
- Setting 定义数据的分布，指定分片数

索引的不同语义：

- 名词：一个 Elasticsearch 集群中，可以创建多个不同的索引
- 动词：保存一个文档到 Elasticsearch 的过程也叫索引 (indexing)
- 名词：一个 B 树索引，一个倒排索引

# 操作集群

