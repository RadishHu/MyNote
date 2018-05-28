# Hive整合HBase

## 应用场景

- 将ETL操作的数据存入HBase
- HBase作为Hive的数据源

## 配置环境

- 修改hive-site.xml文件，添加配置属性(zookeeper的地址)

  ```xml
  <property>      
  <name>hbase.zookeeper.quorum</name>
  <value>node01:2181,node02:2181,node03:2181</value>
  </property>
  ```

- 修改hive-env.sh文件，引入hbase的依赖包

  ```sh
  export HIVE_CLASSPATH=$HIVE_CLASSPATH:/var/local/hbase/lib/*
  ```

## HBase表映射到Hive表

- 在HBase中创建表：表名hbase_test,列族f1、f2、f3

  ```
  create 'hbase_test',{NAME => 'f1',VERSIONS => 1},{NAME => 'f2',VERSIONS => 1},{NAME => 'f3',VERSIONS => 1}
  ```

- 插入数据

  ```
  put 'hbase_test','r1','f1:name','zhangsan'
  put 'hbase_test','r1','f2:age','20'
  put 'hbase_test','r1','f3:sex','male'
  put 'hbase_test','r2','f1:name','lisi'
  put 'hbase_test','r2','f2:age','30'
  put 'hbase_test','r2','f3:sex','female'
  put 'hbase_test','r3','f1:name','wangwu'
  put 'hbase_test','r3','f2:age','40'
  put 'hbase_test','r3','f3:sex','male'
  ```

- 创建基于HBase的hive表

  ```
  CREATE EXTERNAL TABLE hiveFromHbase(
  rowkey string,
  f1 map<STRING,STRING>,
  f2 map<STRING,STRING>,
  f3 map<STRING,STRING>
  ) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
  WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f1:,f2:,f3:")
  TBLPROPERTIES ("hbase.table.name" = "hbase_test");
  ```

  使用外部表映射HBase中的表，这样删除Hive中表，并不会删除HBase中的表

- Hive中查询HBase表

  ```
  select * from hiveFromHbase;
  ```

- Hive中插入数据到HBase表

  ```
  insert into table hiveFromHbase
  SELECT 'r4' AS rowkey,
  map('name','zhaoliu') AS f1,
  map('age','50') AS f2,
  map('sex','male') AS f3
  from person limit 1;
  ```

## Hive表映射到HBase表

- 创建映射HBase的表

  ```
  reate   table hive_test(
  id string,
  name string,
  age int,
  address string
  )STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
  WITH SERDEPROPERTIES ("hbase.columns.mapping" = "
  :key,
  f1:name,
  f2:age,
  f3:address")
  TBLPROPERTIES ("hbase.table.name" = "hbaseFromhive");
  ```

- Hive表加载数据

  ```
  insert overwrite table hive_test select * from hive_source
  ```