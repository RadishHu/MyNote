# MongoDB查询文档

## 语法

find()方法以非结构化的方式显示所有文档

```
db.collection.find(query,projection)
```

- query：可选，使用查询操作符指定查询条件
- projection：可选，使用投影操作符指定返回的键

pretty()方法以格式化的方法来显示所有文档

```
db.collection.find().pretty()
```

## Where语句

| 操作       | 格式                     | 范例                                      | RDBMS中的类似语句  |
| ---------- | ------------------------ | ----------------------------------------- | ------------------ |
| 等于       | {\<key>:\<value>}        | db.col.find({"by":"yutou"}).pretty()      | where by = 'yutou' |
| 小于       | {\<key>:{$lt:\<value>}}  | db.col.find({"likes":{$lt:50}}).pretty()  | where likes < 50   |
| 小于或等于 | {\<key>:{$lte:\<value>}} | db.col.find({"likes":{$lte:50}}).pretty() | where likes <= 50  |
| 大于       | {\<key>:{$gt:\<value>}}  | db.col.find({"likes":{$gt:50}}).pretty()  | where likes > 50   |
| 大于或等于 | {\<key>:{$gte:\<value>}} | db.col.find({"likes":{$gte:50}}).pretty() | where likes >= 50  |
| 不等于     | {\<key>:{$ne:\<value>}}  | db.col.find({"likes":{$ne:50}}).pretty()  | where likes != 50  |

## AND、OR

- AND

  find()方法传入多个key，每个key以逗号隔开，就是SQL的AND条件

  ```
  db.collection.find({key1:value1,key2:value2}).pretty()
  ```

- OR

  OR条件语句使用关键字$or

  ```
  db.collection.find(
  	{
          $or:[
              {key1:value1},{key2,value2}
          ]
  	}
  )
  ```

## Limit与Skip方法

- limit()

  指定从MongoDB中读取的记录条数

  ```
  db.collection.find().limit(NUMBER)
  ```

- skip()

  指定跳过的记录条数

  ```
  db.collection.find().limit(NUMBER).skip(NUMBER)
  ```

## Sort()方法

sort()方法对数据进行排序，可以通过参数指定排序的字段，并是使用1和-1来指定排序的方法，1位升序，-1为降序排列

```
db.collection.find().sort({KEY:1})
```

## MongoDB聚合

使用aggregate()方法来进行聚合计算

```
db.collection.aggregate(AGGREGATE_OPERTION)
```

聚合表达式：

| 表达式    | 描述                                           | 实例                                                         |
| --------- | ---------------------------------------------- | ------------------------------------------------------------ |
| $sum      | 计算总和。                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg      | 计算平均值                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min      | 获取集合中所有文档对应值得最小值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max      | 获取集合中所有文档对应值得最大值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push     | 在结果文档中插入值到一个数组中。               | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet | 在结果文档中插入值到一个数组中，但不创建副本。 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first    | 根据资源文档的排序获取第一个文档数据。         | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last     | 根据资源文档的排序获取最后一个文档数据         | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}]) |

