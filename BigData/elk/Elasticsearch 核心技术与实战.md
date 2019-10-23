# 05 Elasticsearch 的安装与配置

## 安装

- 下载

  ```shell
  wget https://artifacts.elastic.co/downloads/elasticsearch-7.0.0-linux-x86_64.tar.gz
  ```

- 解压

  ```shell
  tar zxvf elasticsearch-7.0.0-linux-x86_64.tar.gz
  ```

- 配置环境变量

  ```shell
  vim /etc/profile
  
  # set jdk path
  export JAVA_HOME=/xxx/jdk1.8.0
  
  #set es path
  export ES_HOME=/xxx/elasticsearch-7.0.0
  export PATH=$PATH:$JAVA_HOME/bin:$ES_HOME/bin
  ```

  使配置文件生效

  ```shell
  source /etc/profile
  ```

- 修改配置文件 - config/elasticsearch.yml

  ```yml
  network.host: ip # 当前主机的 ip
  ```

- 启动

  ```shell
  bin/elasticsearch
  bin/elasticsearch -d # 后台启动
  ```

  启动时的报错解决：

  - org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root

    原因：elasticsearch 不能以 root 用户启动

    解决方案：

    创建一个普通用户

    ```shell
    useradd es
    passwd es
    ```

    修改 elasticsearch 目录的所属用户：

    ```shell
    chown -R es:es $ES_HOME
    ```

    切换到 es 用户，重启 elasticsearch

  - [3] bootstrap checks failed
    [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

    原因：用户最大可创建文件数太小

    解决方案：

    编辑 /etc/security/limits.conf 文件，追加以下内容：

    ```conf
    * soft nofile 65536
    * hard nofile 262144
    * soft nproc 32000
    * hard nproc 32000
    ```

  - \[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

    原因：最大虚拟内存太小

    解决方案：

    编辑 /etc/sysctl.conf 文件，追加以下内容：

    ```conf
    vm.max_map_count=655360
    ```

    执行命令，使配置生效：

    ```shell
    sysctl -p
    ```

  - \[3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured

    解决方案：

    修改 elasticsearch.yml 文件：

    ```yml
    cluster.initial_master_nodes: ["node01"]
    ```

    > node01 为当前节点的主机名

## JVM 配置

- 修改 JVM - config/jvm.options

  7.1 版本默认设置是 1 GB

- 配置建议

  - Xmx 和 Xms 设置成一样
  - Xmx 不要超过机器内存的 50%
  - 不要超过 30 GB - https://www.elastic.co/blog/a-heap-of-trouble

## Web UI

```
node01:9200
```

## 安装与查看插件

查看已经安装的插件：

```
bin/elasticsearch-plugin list
```

安装插件：

```
bin/elasticsearch-plugin install analysis-icu
```

> analysis-icu 是一个国际化的分词插件

通过 web ui 查看已经安装的插件：

```
node01:9200/_cat/plugins
```

## 在开发机运行多个 Elasticsearch 实例

```shell
bin/elasticsearch -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -d
bin/elasticsearch -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -d
bin/elasticsearch -E node.name=node3 -E cluster.name=geektime -E path.data=node3_data -d
```

关闭进程：

```shell
ps | grep elasticsearch

kill pid
```

通过 web ui 查看集群的节点：

```
node01:9200/_cat/nodes
```

# 06 Kibana 的安装

## 安装

- 下载解压 Kibana

- 修改配置文件 - config/kibana.yml

  ```yml
  elasticsearch.url: ["http://node01:9200"]
  ```

  或

  ```yml
  elasticsearch.hosts: ["http://node01:9200"]
  ```

  > 这里需要执行 elasticsearch 的访问地址
  >
  > 使用 *elasticsearch.url* 还是 *elasticsearch.hosts*，以所使用的版本为准

- Web UI

  ```
  node01:5601
  ```

## 工具

- Dashboard

  对数据进行可视化

- Dev Tool

  执行 Elasticsearch 的一些 API

## 插件

- 安装插件

  ```shell
  kibana-plugin install plugin_location
  ```

  > plugin_location，指定插件文件的位置

- 查看安装的插件列表

  ```shell
  kibana-plugin list
  ```

- 移除插件

  ```
  kibana remove
  ```

# 08 Logstash 安装与导入数据



# 09 基本概念：索引、文档和 REST API

## 文档 (Document)

- Elasticsearch 是面向文档的，文档是所有可搜索数据的最小单位

  - 日志文件中的日志项
  - 一部电影的具体信息 / 一张唱片的详细信息
  - MP3 播放器里的一首歌 / 一个 PDF 文档中的具体内容

  可以理解为关系型数据库中的一条数据

- 文档会被序列化为 JSON 格式，保存在 Elasticsearch 中

  - JSON 对象由字段组成
  - 每个字段都有对应的字段类型 (字符串 / 数值 / 布尔 / 日期 / 二进制 / 范围类型)

- 每个文档都有一个 UUID (Unique ID)

  - 可以自己制定 ID
  - 或通过 elasticsearch 自动生成


示例：

将一个保存电影信息的 CSV 文件经过 Logstash 转换后进入 Elasticsearch，就会以 Json 格式保存。

csv 文件

```csv
movieId,title,genres,1,Toy.Story.(1995),Adventure|Animation|Children|Comedy|Fantasy
```

Json 格式

```json
{
    "year":1995,
    "version":"1",
    "genre":[
        "Adventure","Animation",
        "Children","Comedy","Fantasy"
    ],
    "id":"1",
    "title":"Toy Story"
}
```

> Json 文档，格式灵活，不需要预先定义格式
>
> - 字段类型可以指定或通过 Elasticsearch 自动推算
> - 支持数组和嵌套

**文档的元数据**

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

Index 是文档的容器，是一类相似文档的集合

- Index 体现了逻辑空间的概念，每个索引都有自己的 Mapping 定义，用于定义包含的文档的字段名和字段类型
- Shard 体现了物理空间的概念，索引中的数据分散在 Shard 上

索引的 Mapping 和 Settings

- Mapping 定义文档字段的类型
- Setting 定义不同的数据分布，指定用多少分片和数据是怎么分布的

示例：

```json
{
    "movies":{
        "settings":{
            "index":{
                "creation_date":"1552737458543",
                "number_of_shards":"2",
                "number_of_replicas":"0",
                "uuid":"Qnd7lMrNQPGdaeJ9or0tfQ",
                "version":{
                    "created":"6060299"
                },
                "provided_name":"movies"
            }
        }
    }
}
```

**索引的不同语义**

- 名词：一个 Elasticsearch 集群中，可以创建很多个不同的索引
- 动词：保存一个文档到 Elasticsearch 的过程也叫索引 (indexing)
- 名词：一个 B 树索引，一个倒排索引

**Type**

在 7.0 之前，一个 Index 可以设置多个 Types

6.0 开始，Type 已经被废除。7.0 开始，一个索引只能创建一个 Type，也就是 "_doc"。

**Elasticsearch 与 关系型数据库类比**

| RDBMS  | Elasticsearch |
| ------ | ------------- |
| Table  | Index         |
| Row    | Document      |
| Column | Field         |
| Schema | Mapping       |
| SQL    | DSL           |

# 10 基本概念：节点、集群、分片和副本

分布式集群的可用性和扩展性：

- 高可用
  - 服务可用性，允许节点停止服务
  - 数据可用性，部分节点丢失，不会丢失数据
- 可扩展性
  - 请求量 / 数据不多增长，将数据分散到所有节点上

Elasticsearch 的分布式架构：

- 不同的集群通过不同的名字来区分，默认名字 "elasticsearch"
- 通过配置文件修改，或启动时通过 **-E cluster.name=yourName** 进行设定
- 一个集群可以有一个或多个节点

## 节点

节点是一个 Elasticsearch 的实例：

- 本质上是一个 Java 进程
- 一个机器可以运行多个 Elasticsearch 进程，生产环境建议一台机器上只运行一个 Elasticsearch 实例

每一个节点都有名字，通过配置文件配置，或启动时通过 **-E node.name=node1** 指定

每一个节点在启动之后，会分配一个 UID，保存在 data 目录下

**Master-eligible nodes 和 Master Node**

每个节点启动后，默认就是一个 Master eligible 节点，可以通过 **node.master:false** 来禁用

Master-eligible 节点可以参加选主流程，成为 Master 节点

每个节点都保存了集群的状态，只有 Master 节点才能修改集群的状态信息

- 集群状态，维护了一个集群的必要信息：
  - 所有的节点信息
  - 所有的索引和其相关的 Mapping 与 Setting 信息
  - 分片的路由信息

**Data Node 和 Coordinating Node**

- Data Node

  用来保存数据的节点，负责保存分片数据，在数据上起到至关重要的作用

- Coordinating Node

  - 负责接受 Client 的请求，将请求分发到合适的节点，最终把结果汇集到一起
  - 每个节点默认都起到 Coordinating Node 的职责

**其它节点类型**

- Hot & Warm Node

  不同硬件配置的 Data Node，用来实现 Hot & Warm 架构，降低集群部署的成本。Hot 节点是一些配置比较高的节点，Warm 节点配置节点，用来存储一些比较旧的节点

- Machine Learning Node

  负责跑机器学习的 Job，用来做异常检测

- Tribe Node

  Tribe Node 在新的版本中会逐渐被淘汰，5.3 开始使用 Cross Cluster Search。Tribe Node 连接到不同的 Elasticsearch 集群，并支持将这些集群当成一个单独的集群处理

**配置节点类型**

每个节点在启动的时候会读取 **elasticsearch.yml** 文件来决定自己的角色

- 开发环境中一个节点可以承担多种角色
- 生产环境中，应该设置单一的角色的节点

| 节点类型          | 配置参数    | 默认值                                                       |
| ----------------- | ----------- | ------------------------------------------------------------ |
| master eligible   | node.master | true                                                         |
| data              | node.data   | true                                                         |
| ingest            | node.ingest | true                                                         |
| coordinating only | 无          | 每个节点默认都是 coordinating 节点。可以设置其他类型参数为 false |
| machine learning  | node.ml     | true (需要 enable x-pack)                                    |

## 分片

分片分为两类：主分片 (Primary Shard) 和 副分片 (Replica Shard)

- 主分片，可以解决数据水平扩展的问题。通过主分片，可以将数据分布到集群内的所有节点之上
  - 一个主分片是一个运行的 Lucene 的实例
  - 主分片数在索引创建时指定，后续不允许修改，除非 Reindex
- 副本，用于解决数据高可用的问题，副本分片是主分片的拷贝
  - 副本分片，可以动态调整
  - 增加副本数，可以在一定程度上提高服务的可用性 (读取的吞吐)

**分片设定**

对于生产环境中分片的设定，需要提前做好容量规划：

- 分片数设置过小
  - 导致后续无法增加节点来实现水平扩展
  - 单个分片的数据量过大，导致数据重新分配耗时
- 分片数设置多大，7.0 开始，默认主分片设置成 1，解决 over-sharding 的问题
  - 影响搜索结果的相关性打分，影响统计结果的准确性
  - 单个节点上过多的分片，会导致资源浪费，同时也会影响性能

**查看集群的健康状况**

```
GET _cluster/health
```

通过颜色来显示分片的状况：

- Green，主分片和副本都正常分配
- Yellow，主分片全部正常分配，有副本分片未能正常分配
- Red，有主分片为分配

# 11 文档的基本 CRUD 与批量操作

## 文档的 CRUD

**Create 文档**

- 支持自动生成文档 Id 和指定文档 Id 两种方式:

  - 通过调用 **post users/_doc**，系统会自动生成 document id

    ```
    POST my_index/_doc
    {"user":"mike","comment":"You know, for search"}
    ```

  - 使用 **PUT users/_create/1** 创建时，URI 中显示指定 _create，此时如果该 id 的文档已经存在，操作失败

    ```
    PUT my_index/_create/1
    {"user":"mike","comment":"You know, for search"}
    ```

**Get 文档**

```
GET my_index/_doc/1
```

- 找到文档，返回 HTTP 200

  文档元信息：

  - _index 和 _type
  - _version，版本信息，同一个 Id 的文档，即使被删除，Version 号也会不断增加
  - _source，包含文档的所有原始数据

- 找不到文档，返回 HTTP 404

**Index 文档**

Index 和 Create 不一样的地方：如果文档不存在，就索引新的文档，否则，会删除现有的文档，新的文档被索引，版本信息 +1。

```
PUT my_index/_doc/1
{"user":"mike","comment":"You know, for search"}
```

**Update 文档**

Update 不会删除原有的文档，而是更新原有文档的数据

```
POST my_index/_doc/1
{"doc":{"user":"mike", "comment":"You konw, Elasticsearch"}}
```

 **Delete 文档**

```
DELETE my_index/_doc/1
```

## Bulk API

在 Rest 请求中，重新建立网络是非常损耗性能的。Bulk API 可以在一次请求中，对不同的索引进行操作，支持四种类型操作：Index、Create、Update、Delete

当一系列操作中的单条操作失败，不会影响其他操作。并且返回结果中包括每一条执行的结果。

```
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test2", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

## mget

批量读取，减少网络连接所产生的开销，提高性能

```
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_id" : "2"
        }
    ]
}
```

## msearch

批量查询

```
POST kibana_sample_data_ecommerce/_msearch
{}
{"query" : {"match_all" : {}},"size":1}
{"index" : "kibana_sample_data_flights"}
{"query" : {"match_all" : {}},"size":2}
```



## 常见错误返回

| 问题         | 原因               |
| ------------ | ------------------ |
| 404          | 文档没有找到       |
| 无法连接     | 网络故障或集群挂了 |
| 连接无法关闭 | 网络故障或节点出错 |
| 429          | 集群过于繁忙       |
| 4xx          | 请求体格式有错     |
| 500          | 集群内部错误       |

# 12 倒排索引

- 正排索引
- 倒排索引

## 倒排索引的组成

倒排索引包含两部分：

- 单词词典 (Term Dictionary)，记录所有文档的单词，记录单词到倒排列表的关联关系。单词词典一般比较大，可以通过 B+ 树或哈希拉链法实现
- 倒排列表 (Posting List)，记录单词对应的文档结合，由倒排索引项 (Posting) 组成：
  - 文档 ID
  - 词频 TF，该单词在文档中出现的次数，用于相关性评分
  - 位置 (Position)，单词在文档中分词的位置，用于语句搜索 (phrase query)
  - 偏移 (Offset)，记录单词的开始和结束位置，实现高亮显示

## Elasticsearch 的倒排索引

Elasticsearch 的 JSON 文档中每个字段，都有自己的倒排索引

在 Mapping 中可以指定对某些字段不做索引，这样可以节省存储空间，但是那些字段无法用于搜索

# 13 通过 Analyzer 进行分词

## Analysis 与 Analyzer

- Analysis，文本分析，是把全文本转换成一系列单词 (term / token) 的过程，也叫分词
- Analyzer 也叫分词器，Analysis 是通过 Analyzer 来实现的，Elasticsearch 可以使用内置的分词器，也可以定制分词器
- 除了在数据写入时转换词条，匹配 Query 语句是也需要用分析器对查询条件进行分词

## Analyzer 的组成

Analyzer 由三部分组成：

- Character Filters，针对原始文本进行处理，比如去除 html 标签
- Tokenizer，按照一定的规则切分单词
- Token Filter，对切分后的单词进行二级加工，比如转小写

## Elasticsearch 内置的分词器

- Standard Analyzer，默认分词器，按词切分，小写处理
- Simple Analyzer，按照非字母切分 (符号被过滤)，小写处理
- Stop Analyzer，小写处理，停用词过滤 (the, a, is)
- Whitespace Analyzer，按照空格切分，不转小写
- Keyword Analyzer，不分词，直接将输入作为输出
- Patter Analyzer，正则表达式，默认 \W+ (非字符分隔)
- Language，提供 20 多种常见语言的分词器
- Customer Analyzer，自定义分词器

## _analyzer API

用来查看 analyzer 的分词效果，有三种查看方式：

- 直接指定 Analyzer 进行测试

  ```
  Get /_analyzer
  {
  	"analyzer":"standard"
  	"text":"Mastering Elasticsearch, elasticsearch in Action"
  }
  ```

  > "analyzer"，指定分词器的名字

- 指定索引的字段进行测试

  ```
  POST books/_analyze
  {
  	"field":"title",
  	"text":"Mastering Elasticsearch"
  }
  ```

- 自定义分词器

  ```
  POST /_analyze
  {
  	"tokenizer":"standard"
  	"filter":["lowercase"],
  	"text":"Mastering Elasticsearch"
  }
  ```

## 中文分词

中文分词的难点：

- 中文句子，切分成一个个词而不是一个个字
- 英文分词中有自然空格作为分隔
- 一句中文，在不同的上下文，有不同的理解

ICU Analyzer

- 安装 plugin

  Elasticsearch-plugin install analysis-icu

- 提供了 Unicode 的支持，更好的亚洲语言支持

IK

- 支持自定义词库，支持热更新分词字典
- https://github.com/medcl/elasticsearch-analysis-ik

THULAC

- THU Lexucal Analyzer for Chinese，清华大学自然语言处理与社会人文计算实验室的一套中文分词器
- https://github.com/microbun/elasticsearch-thulac-plugin

# 14 Search API 概览

Search API 可以分为两大类：

- URI Search，使用 Http GET 的方式，在 URL 中使用查询参数

- Request Body Search，使用 Elasticsearch 提供的，基于 JSON 格式的更加完备的 Query Domain Specific Language (DSL)

语法：

| 语法                   | 范围                |
| ---------------------- | ------------------- |
| /_search               | 集群上所有的索引    |
| /index1/_search        | index1              |
| /index1,index2/_search | index1 和 index2    |
| /index*/_search        | 以 index 开头的索引 |

## URI 查询

```shell
curl -XGET "http://elasticsearch:9200/kibana_sample_data_ecommerce/_search?q=customer_first_name:Eddie"
```

> q 用来表示查询内容

## Request Body

```shell
curl -XGET "http://elasticsearch:9200/kibana_sample_data_ecommerce/_search" -H 'Content-Type:application/json' -d
{
	"query":{
		"match_all":{}
	}
}
```

> GET，支持 POST 和 GET 请求
>
> kibana_sample_data_ecommerce，索引名
>
> _search，执行搜索操作
>
> query，查询
>
> match_all，返回所有的文档

## 衡量相关性

Information Retrieval

- Pricision (查准率)，尽可能返回较少的无关文档
- Recall (查全率)，尽量返回较多的相关文档
- Ranking，是否能按相关度进行排序

# 15 URI Search 详解

```
GET /movies/_search=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s
{
	"profile":true
}
```

> q，指定查询条件，使用 Query String Syntax
>
> df，默认字段，不指定时，会对所有的字段进行查询
>
> sort，排序
>
> from 和 size，用于分页
>
> Profile，可以查看查询是如何被执行的

**Query String Syntax**

- 指定字段和泛查询

  - 指定字段，q=title:2012 或 q=2012&df=title
  - 泛查询，q=2012 (不指定查询字段)

- Term 和 Phrase

  - Term Query，Beautiful Mind 等效于 Beautiful OR Mind
  - Phrase Query，"Beautiful Mind" 等效于 Beautiful AND Mind，Phrase 查询，要求前后顺序保持一致

- 分组和引号

  - 分组，title:(Beautiful Mind)

    > 对两个单词进行 Term Query 时，需要用括号括起来

  - 引号，title="Beautiful Mind"

- 布尔操作

  AND / OR / NOT  或者 && / || / !

  title:(matrix NOT reloaded)

  > NOT 必须大写

- 分组

  - \+ 表示 must

    ```
    GET /movies/_search?q=title:(Beautiful %2BMind)
    {
    	"profile":"true"
    }
    ```

    > %2B 代表 + 

  - \- 表示 must_not

    title:(+matrix -reloaded)

- 范围查询

  [] 闭区间, {} 开区间

  year:[2019 TO 2019]

  yaar:{* TO 2018}

- 算数符号

  - year:>2010
  - year:(>2010 && <=2018)
  - year:(+>2010 +<=2018)

- 通配符

  ? 代表 1 个字符，* 代表 0 或多个字符

  通配符查询效率低，占用内存大，不建议使用，特别是放在最前面

  title:mi?d

  title:be*

- 正则表达

  title:[bt]oy

- 模糊匹配与近似匹配

  - title:befutifl~1
  - title:"lord rings"~2



# 24 基于此项和基于全文的搜索

## 基于 Term 的查询

Term 的重要性：

- Term 是表达语义的最小单位，搜索和利用统计语言模型进行自然语言处理都需要处理 Term

特点：

- Term Level Query: Term Query / Range Query / Exists Query / Prefix Query / Wildcard Query
- ES 中， Term 查询，对输入不做分词，会将输入作为一个整体，在倒排索引中查找准确的词项，并且使用相关度算分公式为每个包含该词项的文档进行相关度算分
- 可以通过 Constant Score 将查询转换成一个 Filtering，避免算分，并利用缓存，提高性能

## 基于全文的查询

基于全文本查：Match Query / Match Phrase Query / Query String Query

特点：

- 索引和搜索时都会进行分词，查询字符串先传递到一个合适的分词器，然后生成一个供查询的词项列表
- 查询时，先回对输入的查询进行分词，然后每个词逐个进行底层的查询，最终将结果进行合并，并未每个文档生成一个算分