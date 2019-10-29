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

## Document CRUD API

### Index API

添加一个 id 为 1 的文档到 "twitter" 索引中：

```
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

index 操作返回的结果为：

```json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result" : "created"
}
```

> `_shard` 提供了副本分片的处理结果

**自动创建索引**

如果在写入文档前没有创建索引，那么会根据 `index templates` 自动创建索引，如果索引没有指定 `mapping`，那也会自动创建。

通过设置 `action.auto_create_index` 为 false 来禁止自动创建索引，默认为 true。可以设置为通过逗号分隔的列表，允许自动创建符合规则的索引。通过 `+` 或 `-` 前缀来指定允许/禁止自动创建的索引。

```
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*" 
    }
}

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "false" 
    }
}

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true" 
    }
}
```

**选择操作类型**

可以通过 `op_type` 来指定操作类型为 `create`，这样如果添加文档 id 已经存在，那操作会失败：

```
PUT twitter/_doc/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

也可以在 uri 中指定操作类型：

```
PUT twitter/_create/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

**自动生成 id**

如果添加文档时没有指定 id, 会自动给文档生成 id，`op_type` 会自动设置为 `create` 操作：

```
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

> 这里使用的 **POST** 而不是 **PUT**

index 操作返回的结果：

```json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "W0tpsmIBdwcYyG50zbta",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result": "created"
}
```

**路由**

默认情况下，新增的文档被分配哪个分片是通过对文档的 id 进行 Hash 来决定的。如果要进行精确的控制，可以通过 `routing` 参数指定进行 Hash 的值：

```
POST twitter/_doc?routing=kimchy
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

> 文档会根据 `routing` 参数指定的 "kimchy" 来决定写入哪个分片

可以在 mapping 中通过 `_routing` 字段来指定通过文档中的字段进行 Hash，这个会带来很小的解析文档的开销。如果 `_routing` 字段被指定为 required，那么在写入文档时必须指定 `routing` 参数，否则会写入失败。

**等待可用的分片**

在写入文档时，可以指定分片数，只有可用的分片数达到指定的数量才会写入文档。如果可用的分片数达不到指定的数量，写操作必须等到或重试，直到启动分片或达到超时时间。默认情况下，只要主分片可用，写操作就可以执行 (`wait_for_active_shards=1`)。这个设置可以通过 `index.write.wait_for_active_shards` 进行动态的修改。如果要在所有的写操作中都启动这个配置，可以修改 `wait_for_active_shards` 参数。

这个参数的值也可以设置为 `all` 来指定写操作时所有的分片都可用 (`number_of_replicas + 1`)。

**超时时间**

执行写入文档操作的主分片可能不可用，默认情况写，写操作会等主分片恢复过来直到 1 分钟，会报写操作失败，可以通过 `timeout` 参数来指定超时时间：

```
PUT twitter/_doc/1?timeout=5m
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

**写入版本号**

每个文档都有一个版本号，默认情况下版本号会从 1 开始，在每次更新时 (包括删除操作) 递增。这个版本号可以手动指定，设置 `version_type` 为 `external`，设置的版本号必须为 Long 型正数字 (0 - 9.2e+18)。

手动设置版本号时，系统会检测设置的版本号是否比当前保存的版本号大，如果比当前版本好大，则文档会被保存并使用新的版本号；如果指定的版本号比当前保存的版本号小，会出现版本冲突，写入操作会失败：

```
PUT twitter/_doc/1?version=2&version_type=external
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
```

`version_type` 可选的参数：

- `internal`

  只有指定的版本号与当前保存的版本号相同时，才会写入文档

- `external` / `exterval_gt`

  只有指定的版本号比当前保存的版本号大或第一次写入时才会写入成功

- external_gte

  只有指定的版本号大于等于当前保存的版本号或第一次写入时才会写入成功

### Get API

通过 Get API 可以通过 id 从索引库中获取一个 JSON 文档。下面示例从 "twitter" 索引库中获取 id 为 0  的文档：

```
GET twitter/_doc/0
```

get 操作的执行结果：

```json
{
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "0",
    "_version" : 1,
    "_seq_no" : 10,
    "_primary_term" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

Get API 也可以用来验测文档是否存在：

```
HEAD twitter/_doc/0
```

**实时性**

默认情况下，Get API 是实时的，并且不会受 index 操作刷新频率影响。如果一个文档已经被更新，但是还没有刷新，Get API 会调用一个刷新操作来使文档可访问，这个也会使其它的文档也被刷新。如果要禁用 Get 操作的实时性，可以设置 `realtime` 参数为 `false`。

**过滤 Source**

默认情况下，Get 操作会返回 `_source` 字段的全部内容，除非使用 `stored_fields` 参数或 禁用`_source` 字段。可以通过 `_source` 参数来关闭 `_source`:

```
GET twitter/_doc/0?_source=false
```

如果只需要 `_source` 中的一两个字段，可以使用 `_source_includes` 和 `_source_excludes` 参数，这两个参数可以指定一个逗号分隔的列表，也可以使用通配符：

```
GET twitter/_doc/0?_source_includes=*.id&_source_excludes=entities
```

如果只是要指定要包含的字段，可以使用下面这个表达式：

```
GET twitter/_doc/0?_source=*.id,retweeted
```

**Stored Fields**

Get 操作可以通过 `stored_field` 指定一个请求返回 stored field 集合，如果请求的字段没有被保存，那这个字段会被忽略掉。示例如下：

创建一个索引库：

```
PUT twitter
{
   "mappings": {
       "properties": {
          "counter": {
             "type": "integer",
             "store": false
          },
          "tags": {
             "type": "keyword",
             "store": true
          }
       }
   }
}
```

添加一个文档：

```
PUT twitter/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

Get 操作返回的结果：

```json
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "1",
   "_version": 1,
   "_seq_no" : 22,
   "_primary_term" : 1,
   "found": true,
   "fields": {
      "tags": [
         "red"
      ]
   }
}
```

> 从文档中取回的对应字段的值通常以数组返回。因为 `counter` 字段没有被保存 get 请求在获取 `stored_fields` 时忽略了它。

**只获取 _source 内容**

可以通过 `/{index}/_source/{id}` 的形式类是获取文档 `_source` 字段中的内容：

```
GET twitter/_source/1
```

也可以使用 source filter 参数来控制返回的字段：

```
GET twitter/_source/1/?_source_includes=*.id&_source_excludes=entities
```

可以通过 HEAD 请求来判断文档 `_source` 是否存在：

```
HEAD twitter/_source/1
```

**Routing**

如果保存文档时使用 routing 来控制文档保存的分片，那么在获取文档时，也应该指定 routing 字段：

```
GET twitter/_doc/2?routing=user1
```

**副本分片优先级**

通过 `preperence` 来控制哪个副本分片来执行 get 请求，默认情况下是随机分发的。`preference` 可以设置为一下参数：

- _local，如果可能话，get 请求会在本地分配的分片上执行
- custom string value，自定义值用于确保将相同的分片用于相同的自定义值

### Delete API

Delete API 可以通过 Id 从索引库中删除一个 JSON 文档。下面示例从 `twitter` 索引库删除 id 为 1 的 JSON 文档：

```
DELETE /twitter/_doc/1
```

Delete 操作的执行结果如下：

```json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 2,
    "_primary_term": 1,
    "_seq_no": 5,
    "result": "deleted"
}
```

### Delete By Query API

`_delete_by_query` 会删除匹配到的所有文档：

```
POST twitter/_delete_by_query
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}
```

这个操作返回的结果为：

```json
{
  "took" : 147,
  "timed_out": false,
  "deleted": 119,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 119,
  "failures" : [ ]
}
```

> - took，整个操作消耗的时间，单位毫秒
> - timed_out，在删除过程中执行的任何请求如果超时，那这个值会返回 `true`
> - total，成功处理的文档个数
> - deleted，成功删除的文档个数
> - batches，通过 `delete_by_query` 操作回滚的响应数
> - version_conflicts，`delete_by_query` 操作中版本冲突的个数
> - noops，在 `delete_by_query` 操作中，这个字段的值为 0
> - retries，`delete_by_query` 操作中重试的次数，`bulk` 是 bulk 操作重试的次数，`search` 是 search 操作重试的次数
> - throttled_mills，为了遵从 `request_per_second` 参数设置的值，请求过程中休眠的时间，单位毫秒
> - requests_per_second，每秒钟执行的有效请求次数
> - throttled_until_millis，在`_delete_by_query` 操作中这个字段值始终为 0
> - failures，在处理过程中所有不可恢复的报错

### Update API

**通过脚本更新文档**

Update API 通过脚本来更新一个文档，这个操作从索引库获取文档，并运行脚本，最后返回处理的结果。这个操作意味着对文档进行重新索引，只是它减少了网络的开销，并减少了跟 get 和 index 操作冲突的可能性。示例如下：

先添加一个文档：

```
PUT test/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

通过脚本增加 counter 字段的值：

```
POST test/_update/1
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
```

给 tags 字段中添加一个值：

```
POST test/_update/1
{
    "script" : {
        "source": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params" : {
            "tag" : "blue"
        }
    }
}
```

移除 tags 字段中的值：

```
POST test/_update/1
{
    "script" : {
        "source": "if (ctx._source.tags.contains(params.tag)) { ctx._source.tags.remove(ctx._source.tags.indexOf(params.tag)) }",
        "lang": "painless",
        "params" : {
            "tag" : "blue"
        }
    }
}
```

出列 `_source` 外，以下的值也可以在 `ctx` 中指定：`_index`, `_type`, `_id`, `_version`, `_routing` 和 `_now`。

也可以给文档添加新的字段：

```
POST test/_update/1
{
    "script" : "ctx._source.new_field = 'value_of_new_field'"
}
```

移除文档中的字段：

```
POST test/_update/1
{
    "script" : "ctx._source.remove('new_field')"
}
```

在更新过程中添加条件判断：

```
POST test/_update/1
{
    "script" : {
        "source": "if (ctx._source.tags.contains(params.tag)) { ctx.op = 'delete' } else { ctx.op = 'none' }",
        "lang": "painless",
        "params" : {
            "tag" : "green"
        }
    }
}
```

> 如果 `tags` 字段中包含 green，则删除这个文档，如果不包含就什么操作都不执行

**通过部分文档更新**

Update API 也支持通过传参部分文档来更新文档，这部分文档会合并近已存在的文档中。如果要完全更新一个文档，可以使用 `index` API。下面示例在一个已经存在的文档中添加一个字段：

```
POST test/_update/1
{
    "doc" : {
        "name" : "new_name"
    }
}
```

> 如果 `doc` 和 `script` 都被指定了，`doc` 会被忽略

### Update By Query API

`_update_by_query` 操作最简单的用法是在不改变 source 的情况下更新所有的文档：

```
POST twitter/_update_by_query?conflicts=proceed
```

执行的结果如下：

```json
{
  "took" : 147,
  "timed_out": false,
  "updated": 120,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 120,
  "failures" : [ ]
}
```

也可以通过 `query` 限制 `_update_by_query` 操作的执行范围，一下示例更新 `twitter` 索引库的 `user` 字段为 `kimchy` 的文档：

```
POST twitter/_update_by_query?conflicts=proceed
{
  "query": { 
    "term": {
      "user": "kimchy"
    }
  }
}
```

`_update_by_query` 操作也支持通过脚本更新文档，下面的示例增加 kimchy tweets 的 likes 字段的值：

```
POST twitter/_update_by_query
{
  "script": {
    "source": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

### Multi Get API

Multi Get API 会根据 index、type、id 等信息获取多个文档。请求示例：

```
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

可以在请求 uri 中加入索引，这样就不需要在请求体中指定索引：

```
GET /test/_mget
{
    "docs" : [
        {
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

type 也可以添加到请求 的 uri 中：

```
GET /test/_doc/_mget
{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}
```

在这种情况下可以使用 `ids` ：

```
GET /test/_doc/_mget
{
    "ids" : ["1", "2"]
}
```

### Bulk API

通过 bulk API 可以在一次请求中执行多个操作，这些操作可以是 `index`、 `create`、 `delete` 和 `update`。请求示例：

```
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

> `index` 和 `create` 操作需要在下一行添加 source

请求执行的结果如下：

```json
{
   "took": 30,
   "errors": false,
   "items": [
      {
         "index": {
            "_index": "test",
            "_type": "_doc",
            "_id": "1",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 0,
            "_primary_term": 1
         }
      },
      {
         "delete": {
            "_index": "test",
            "_type": "_doc",
            "_id": "2",
            "_version": 1,
            "result": "not_found",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 404,
            "_seq_no" : 1,
            "_primary_term" : 2
         }
      },
      {
         "create": {
            "_index": "test",
            "_type": "_doc",
            "_id": "3",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 2,
            "_primary_term" : 3
         }
      },
      {
         "update": {
            "_index": "test",
            "_type": "_doc",
            "_id": "1",
            "_version": 2,
            "result": "updated",
            "_shards": {
                "total": 2,
                "successful": 1,
                "failed": 0
            },
            "status": 200,
            "_seq_no" : 3,
            "_primary_term" : 4
         }
      }
   ]
}
```

### Reindex API

Reindex 需要随用库中所有文档都开启 `_source` ，reindex 操作不会自动创建目标索引库，因此需要在执行 reindex 操作前先创建索引库，包括 mapping、分片数、副本分片数等。

Reindex 最基本的操作是从一个索引库复制文档到另一个索引库：

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

这个请求返回的结果如下：

```json
{
  "took" : 147,
  "timed_out": false,
  "created": 120,
  "updated": 0,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 120,
  "failures" : [ ]
}
```

设置 `op_type` 为 `create`，reindex 操作只会创建目标索引库中缺少的文档，如果文档已经存在会造成版本冲突：

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  }
}
```

可以只把源索引库中的部分数据拷贝到目标索引库：

```
POST _reindex
{
  "source": {
    "index": "twitter",
    "query": {
      "term": {
        "user": "kimchy"
      }
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

可以通过列表来指定多个源索引库：

```
POST _reindex
{
  "source": {
    "index": ["twitter", "blog"]
  },
  "dest": {
    "index": "all_together"
  }
}
```

## Cat API

Cat API 的通用参数：

- Verbose

  在请求 URL 后添加 `v` ，给返回数据添加头部信息，显示每列的含义：

  ```
  GET /_cat/master?v
  ```

- help

  在请求 URI 后添加 `help`，返回每列含义：

  ```
  id   |   | node id
  host | h | host name
  ip   |   | ip address
  node | n | node name
  ```

### cat aliases

`aliases` 命令查询索引库的别名，包括 filter 和 routing 信息：

```
GET /_cat/aliases?v
```

命令返回结果：

```
alias  index filter routing.index routing.search
alias1 test1 -      -            -
alias2 test1 *      -            -
alias3 test1 -      1            1
alias4 test1 -      2            1,2
```

如果只想获取指定别名的信息，可以在 URL 中添加逗号分隔的列表：

```
GET /_cat/aliases/alias1,alias2
```

### cat allocation

`allocation` 命令查询每个数据节点上的分片数和磁盘空间：

```
GET /_cat/allocation?v
```

命令返回结果：

```
shards disk.indices disk.used disk.avail disk.total disk.percent host      ip      node
     1       260b    47.3gb     43.4gb    100.7gb         46 127.0.0.1  127.0.0.1 CSUXak2
```

### cat count

`count` 命令查询这个集群的文档个数：

```
GET /_cat/count?v
```

命令返回结果：

```
epoch      timestamp count
1475868259 15:24:19  121
```

也可以获取单个索引库的文档个数：

```
GET /_cat/count/twitter?v
```

### cat fielddata

`fielddata` 命令查询每个数据节点中 fielddata 占用的堆内存：

```
GET /_cat/fielddata?v
```

命令返回的结果：

```
id                     host      ip        node    field   size
Nqk-6inXQq-OxUfOUI8jNQ 127.0.0.1 127.0.0.1 Nqk-6in body    544b
Nqk-6inXQq-OxUfOUI8jNQ 127.0.0.1 127.0.0.1 Nqk-6in soul    480b
```

可以在 URL 中通过逗号分隔的列表指定要查看的 fields:

```
GET /_cat/fielddata?v&fields=body,soul?v
```

### cat health

`health` 是 `_cluster/health` 命令的简写：

```
GET /_cat/health?v
```

命令返回结果如下：

```
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1475871424 16:17:04  elasticsearch green           1         1      1   1    0    0        0             0                  -                100.0%
```

添加 `ts` 参数来禁止 timestamping 的输出：

```
GET /_cat/health?v&ts=false
```

### cat indices

`indices` 命令查询索引库的信息：

```
GET /_cat/indices/?v
```

命令返回结果如下：

```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   twitter  u8FNjxh8Rfy_awN11oDKYQ   1   1       1200            0     88.1kb         88.1kb
green  open   twitter2 nYFWZEO7TUiOjLQXBaYJpA   1   0          0            0       260b           260b
```

`indices` 命令可添加一些参数：

- 查询所有状态为 yellow 的索引库

  ```
  GET /_cat/indices?v&health=yellow
  ```

- 根据索引中文档的个数进行排序

  ```
  GET /_cat/indices?v&s=docs.count:desc
  ```

- 查询索引库合并操作的次数

  ```
  GET /_cat/indices/indexName?pri&v&h=health,index,pri,rep,docs.count,mt
  ```

- 查询每个索引库占用的内存

  ```
  GET /_cat/indices?v&h=i,tm&s=tm:desc
  ```

  







