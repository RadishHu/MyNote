# Hive-UDF

UDF，User-defined Function，自定义函数

UDF有三种：

- 普通UDF，User-defined Function

  操作单行数据，输出一个数据

- 聚合UDF，UDAF，User-defined Aggregate Function

  接收多行数据，输出一个数据

- 表生成UDF，UDTF，User-defined  Table-generating Function

  接收一行数据，输出一个表

## 编写UDF

- 一个UDF必须继承`org.apache.hadoop.hive.ql.exec.UDF`

- 一个UDF至少要实现`evaluate()`方法

  > UDF名对大小写不敏感

## 编写UDAF

一个UDAF计算函数必须实现5个方法：

- init()方法，init()方法负责初始化计算函数并重设它的内部状态