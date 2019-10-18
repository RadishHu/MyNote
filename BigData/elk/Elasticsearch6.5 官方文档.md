# Elasticsearch Reference

## 探索集群

### 集群健康

```
GET _cat/health?v
```

返回值：

```
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```

> epoch，时间戳
>
> relo，relocating_shards，重新安置的分片
>
> init，initializing_shards，初始化的分片
>
> unassign，unassigned_shards，未分配副本分片

**查看节点信息：**

```
GET _cat/nodes?v
```

### 查看所有的 Index

```
GET /_cat/indices?v
```

### 创建 Index

创建一个名为 "customer" 的 Index:

```
PUT /customer?pretty
```

> 追加 `pretty`，以格式化的形式打印 JSON 响应

### 读写 Document

写 Document

```
PUT /customer/_doc/1?pretty
{
	"name":"John Doe"
}
```

> - 如果写入的 Index 不存在，会自动创建 Index
> - 如果写入的这个 Document 已经存在，会删除已经存在的 Document，然后创建一个新的 Document

写 Document

```
GET /customer/_doc/1?pretty
```

### 删除 Index

```
DELETE customer?pretty
```

## 数据操作

### 更新 Document

修改字段的值：

```
POST /customer/_doc/1/_update?pretty
{
	"doc":{"name": "Jane Doe"}
}
```

添加字段：

```
POST /customer/_doc/1/_update?pretty
{
	"doc":{"name": "Jane Doe", "age": 20}
}
```

更新时也可以使用一些简单的脚本，增加年龄：

```
POST /customer/_doc/1/_update?pretty
{
	"script":{"ctx._source.age += 5"}
}
```

> `ctx._source` 指向当前要更新的 document

### 删除 Document

```
DELETE /customer/_doc/2?pretty
```

### 批量操作

可以使用 `_bulk` 进行批量操作，它通过尽可能少的网络 IO 来进行多个操作

创建两个 documents:

```
POST /customer/_doc/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```

更新第一个 document，删除第二个 document:

```
POST /customer/_doc/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```

Bulk API  如果其中一个失败，其它的还会继续执行。

## 探索数据

### Search API

有两种方式运行 Search API：REST request URI 和 REST request body

- REST request URI

  ```
  GET /bank/_search?q=*&sort=account_number:asc&pretty
  ```

  > q=，指定查询条件
  >
  > sort=account_number:asc，指定排序字段

  请求响应：

  ```json
  {
    "took" : 63,
    "timed_out" : false,
    "_shards" : {
      "total" : 5,
      "successful" : 5,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : 1000,
      "max_score" : null,
      "hits" : [ {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "0",
        "sort": [0],
        "_score" : null,
        "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
      }, {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "1",
        "sort": [1],
        "_score" : null,
        "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
      }, ...
      ]
    }
  }
  ```

  > took，查询所花费的时间，单位 ms
  >
  > timed_out，查询是否超时
  >
  > _shards，查询分片的数量，成功和失败分片的数量
  >
  > hits，查询结果
  >
  > hits.total，查询到 document 的数量
  >
  > hits.hits，查询结果，默认显示前 10 条
  >
  > hits.sort，排序编号，如果通过 score 排序，不会有这项

- REST request body

  ```
  GET /bank/_search
  {
    "query": { "match_all": {} },
    "sort": [
      { "account_number": "asc" }
    ]
  }
  ```

### Query Language

```
GET /bank/_search
{
	"query":{"match_all": {}}
}
```

> `match_all`，运行的 query 类型，搜索指定 index 中的所有 document

除了 `query` ，我们还可以指定其它的参数：

- size

  指定返回结果的个数，默认为10

  ```
  GET /bank/_search
  {
    "query": { "match_all": {} },
    "size": 1
  }
  ```

- from

  指定返回结果开始的偏移量

  ```
  GET /bank/_search
  {
    "query": { "match_all": {} },
    "from": 10,
    "size": 10
  }
  ```

  > 返回 10-19 的 document，偏移量从 0 开始

- sort

  指定排序字段

  ```
  GET /bank/_search
  {
    "query": { "match_all": {} },
    "sort": { "balance": { "order": "desc" } }
  }
  ```



# Mapping

每个字段可以为一下的数据类型：

- 简单的类型：text, keyword, date, long, double, boolean, ip
- 支持嵌套 JSON 的类型：object, nested
- 特定的类型：geo_point, geo_shape, completion

可以在创建 Index 时指定 mapping:

```
PUT my_index 
{
  "mappings": {
    "_doc": { 
      "properties": { 
        "title":    { "type": "text"  }, 
        "name":     { "type": "text"  }, 
        "age":      { "type": "integer" },  
        "created":  {
          "type":   "date", 
          "format": "strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}
```

> my_index，创建 index 的名字
>
> doc，mapping 的类型
>
> properties，指定字段
>
> format，字段可以有两种格式

## 字段数据类型

### 基本数据类型

- string

  text, keyword

- 数值类型

  logn, integer, short, byte, double, float, half_float, scaled_float

- 日期类型

  date

- Boolean 类型

  boolean

- 二进制类型

  binary

- 范围类型

  integer_range, float_range, long_range, double_range, date_range

  