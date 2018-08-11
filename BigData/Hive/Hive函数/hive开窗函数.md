# Hive开窗函数

参考：[Hive 开窗函数](https://blog.csdn.net/wangpei1949/article/details/81437574)

## 目录

> * [简介](#chapter1)
> * [over()使用](#chapter2)
> * [聚合开窗函数](#chapter3)
> * [排序开窗函数](#chapter4)

## 简介 <a id="chapter1"></a>

聚合函数的行集是组，每组(gourp by)返回一个值，开窗函数聚合的行集是窗口

开窗函数在聚合函数后增加一个`OVER`关键字，开窗函数的格式：

```
聚合函数(列) OVER(选项)
```

> 当选项为空时，开窗函数会对结果集中的所有行进行聚合计算

## over()使用方法 <a id="chapter2"></a>

over中可以使用的开窗函数：

- over(order by filder)，按照filder进行排序，order by是个默认的开窗函数
- over(partition by filder)，按照filder分区
- over(partition by filder1 order by filder2)，按照filder1分区，在一个分区内按照filder2排序

开窗的窗口范围：

- over(order by filder1 range between 2 preceding and 2 following)：窗口范围为当前数据减2-加2的范围

  示例：

  sum(filder) over(order by filder between 2 preceding and 2 following)：表示对filder-2 ——filder+2范围内的值进行求和

- over(order by salary rows between 2 preceding and 2 following)：窗口范围为当前行前后各移动2行

- 窗口不做限制：

  over(order by filder range between unbounded preceding and unbounded following)

  over(order by filder rows between unbounded preceding and unbounded following)

## 聚合开窗函数 <a id="chapter3"></a>

### 测试数据

```
-- 建表
create table student_scores(
id int,
studentId int,
language int,
math int,
english int,
classId string,
departmentId string
);
-- 写入数据
insert into table student_scores values 
  (1,111,68,69,90,'class1','department1'),
  (2,112,73,80,96,'class1','department1'),
  (3,113,90,74,75,'class1','department1'),
  (4,114,89,94,93,'class1','department1'),
  (5,115,99,93,89,'class1','department1'),
  (6,121,96,74,79,'class2','department1'),
  (7,122,89,86,85,'class2','department1'),
  (8,123,70,78,61,'class2','department1'),
  (9,124,76,70,76,'class2','department1'),
  (10,211,89,93,60,'class1','department2'),
  (11,212,76,83,75,'class1','department2'),
  (12,213,71,94,90,'class1','department2'),
  (13,214,94,94,66,'class1','department2'),
  (14,215,84,82,73,'class1','department2'),
  (15,216,85,74,93,'class1','department2'),
  (16,221,77,99,61,'class2','department2'),
  (17,222,80,78,96,'class2','department2'),
  (18,223,79,74,96,'class2','department2'),
  (19,224,75,80,78,'class2','department2'),
  (20,225,82,85,63,'class2','department2');
```

- count

- sum

- min

- max

- avg

- first_value

  返回分区窗口中的第一个值

- last_value

  返回分区窗口中的最后一个值

- lag

  统计当前行往上第n个值，语法：

  ```
  lag(col,n,default)
  ```

  > col：列名
  >
  > n：往上n行
  >
  > default：当往上第n行为NULL时，取默认值，不指定取NULL

- lead

  统计当前行往后的第n个值，语法：

  ```
  lead(col,n,default)
  ```

- cume_dist

  计算某个值在窗口中的累计分布，生序排序时，累计分布的计算公式：

  小于当前值的数量 / 窗口内的总行数

## 排序开窗函数 <a id="chapter4"><a>

- rank

  rank函数基于over()函数中 order by 确定一个值的排名，当出现相同排名的时候，排名可能是不连续的

- dense_rank

  dense_rank与rank的区别在于：当出现排名相同的时候，dense_rank接下来的排名是连续的

- ntile

  将窗口中的已排序的行按指定的数量，划分为数量尽可能相等的组，并返回它所在组的排名，语法：

  ```
  ntile(2) over(partition by field1 order by field2) as ntileValue
  ```

  > 把排序后的行平分为两组

- row_number

  对窗口内的数据进行排序，返回当前值的排名，从1开始

- percent_rank

  计算当前行的百分比排名，计算公式：

  (当前行rank值-1) / (窗口内总行数-1)

