# HIVE_JOIN调优

Hive支持通常的通常的SQL JOIN，但是只支持等值连接

## 目录

> * [生成MR job](#chapter1)
> * [表连接顺序优化](#chapter2)
> * [OUT JOIN优化](#chapter3)
> * [LEFT SEMI JOIN](#chapter4)
> * [Map Side Join](#chapter5)

## 生成MR job <a id="chapter1"></a>

- 生成一个MR job

  多表连接，如果多个表是使用同一个字段进行关联，则只会生成一个MR job：

  ```sql
  SELECT a.val, b.val, c.val FROM a JOIN b ON (a.key = b.key1) JOIN c ON (c.key = b.key1)
  ```

  > 两次join都使用了b.key1

- 生成多个MR job

  多表连接，如果多个表不是使用同一个字段进行关联，则会生成多个MR job：

  ```sql
  SELECT a.val, b.val, c.val FROM a JOIN b ON (a.key = b.key1) JOIN c ON (c.key = b.key2)
  ```

## 表连接顺序优化 <a id="chapter2"></a>

多表连接，会转换成多个MR job，每个MR job在Hive中成为Stage。Hive假定join中的最后一个表是最大的那个表，因为join前一阶段生成的数据会存在于Reduce的buffer中，与后面的表join的时候，直接从buffer中读取缓存的中间结果数据，与大表中的key进行连接，这样速度会更快，也尽可能避免内存缓冲区溢出

```sql
SELECT a.val, b.val, c.val FROM a JOIN b ON (a.key = b.key1) JOIN c ON (c.key = b.key1)
```

> 在这个查询中，数据量应该是：a < b < c

通过Hint信息来指定将哪个表作为大表：

```sql
SELECT /*+ STREAMTABLE(a) */ a.val, b.val, c.val FROM a JOIN b ON (a.key = b.key1) JOIN c ON (c.key = b.key1)
```

> 将a表被视为大表，因此会对表b和c进行join，然后将得到的结果与表a进行join

## OUTER JOIN优化 <a id="chapter3"></a>

- 将where条件放在on中

  ```sql
  SELECT a.val, b.val FROM a LEFT OUTER JOIN b ON (a.key=b.key)
  WHERE a.ds='2009-07-07' AND b.ds='2009-07-07'
  ```

  > 执行顺序：先进行两表的关联查询，然后再使用where条件进行过滤，

  可以进行优化，把where条件放在where后边：

  ```sql
  SELECT a.val, b.val FROM a LEFT OUTER JOIN b
  ON (a.key=b.key AND b.ds='2009-07-07' AND a.ds='2009-07-07')
  ```

- 增加分区过滤

  ```sql
  SELECT a.val, b.val FROM a LEFT OUTER JOIN b ON (a.key=b.key)
  WHERE a.ds='2009-07-07' AND b.ds='2009-07-07' AND a.par='aaa' AND b.par='aaa'
  ```

  > 这样增加分区过滤会改变查询结果，查询结果会与INNER JOIN一样，因为OUTER JOIN结果会出现为NULL的情况，这样添加分区过滤，会过滤掉为NULL的数据

  考虑把分区过滤条件添加到ON条件中，对于OUTER JOIN是不行的，分区过滤条件会被忽略掉，对于INNER JOIN是起作用的

  使用嵌套SELECT语句：

  ```sql
  SELECT a.val,b.val FROM 
  (SELECT a.val FROM a WHERE a.ds='2009-07-07' AND a.par='aaa') aa
  LEFT OUTER JOIN
  (SELECT b.val FROM b WHERE b.da='2009-07-07' AND b.par='aaa') bb
  ON aa.key=bb.key
  ```

## LEFT SEMI JOIN <a id="chapter4"></a>

LEFR SEMI JOIN只返回左边表中符合ON语句判定条件的记录，这个对INNER JOIN的一个优化。在SQL中，可以通过IN...EXISTS来实现：

```sql
SELECT a.yml FROM a
WHERE a.yml IN
(SELECT d.yml FROM b);
```

Hive中不支持这种查询，但是可以通过LEFT SEMI JOIN实现：

```sql
SELECT a.yml FROM a
LEFT SEMI JOIN b ON a.yml = b.yml
```

> SELECT 和 WHERE语句中不能引用右边表的字段
>
> LEFT SEMI JOIN左表中一条记录，在右表中一旦找到匹配的记录，就会停止扫描，因此效率比较高
>
> Hive不支持RIGHT SEMI JOIN

## Map Side Join <a id="chapter5"></a>

当关联表中存在一张小表，可以将小表完全存放在内存中，这样在Map阶段，就可以将大表数据与小表匹配，从而省略了reduce过程

- 通过Hint触发Map Join

  ```sql
  SELECT /*+ MAPJOIN(b) */ a.key, a.value FROM a JOIN b ON a.key = b.key
  ```

  > 将表b放在内存中

- 通过修改属性

  ```
  hive> set hive.auto.convert.join=true;
  ```

  > 这个属性值默认为false

  配置默认使用map side join小表的大小：

  ```
  hive.mapjoin.samlltable.filesize=25000000
  ```

  如果希望自动启动这个优化的话，可以将这两个属性设置在`$HOME/.hiverc`文件中

  hive对与RIGHT OUTER JOIN 和FULL OUTER JOIN不支持这个优化

