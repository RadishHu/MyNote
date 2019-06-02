# PySpark



## Spark Shell on Yarn

当 Spark 以 `on Yarn` 的模式运行时，启动 `spark-shell` 和 `pyspark` 会报错 `Error: Cluster deploy mode is not compatible with master "yarn-client"` ，解决方式：

```shell
spark2-shell --master yarn --deploy-mode client
pyspark2 --master yarn --deploy-mode client
```



## SparkSession

创建 SparkSession 时会报错 `TypeError: 'Builder' object is not callable` 

```python
spark = SparkSession\
        .builder()\
        .appName("SparkSessionZipsExample")\
        .config("spark.sql.warehouse.dir", warehouseLocation)\
        .enableHiveSupport()\
        .getOrCreate()
```

解决方式：把 `builder()` 换成 `builder`



```shell
spark2-submit sparkTest.py --master yarn --deployMode cluster
```



## DataFrame

保存到 hive 表中

```
df.write.partitionBy("key").format("hive").mode("append").saveAsTable("hive_table")
```

参考：https://issues.apache.org/jira/browse/SPARK-18426