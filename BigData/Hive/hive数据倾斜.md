# hive数据倾斜

## 数据倾斜的原因

- key分布不均匀
- 业务数据本身的特性
- SQL语句本身就存在数据倾斜

## 解决方案

- 大小表关联，小表key比较集中：

  采用map side join，将小表存放在内存中，在map阶段完成join

- 大表join大表，key中的0或空值过多：

  把空值的key变成一个字符串加上随机数，把倾斜的数据分发到不同的reduce上

- count(distinct)语句产生的数据倾斜：

  使用sum() group by 替换count(distinct)

- 业务数据本身存在倾斜：

  首先考虑业务逻辑优化，如果效果不明显，可以将倾斜的数据单独拿出来处理