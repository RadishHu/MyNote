# Spark Streaming

Spark Streaming的工作流程是将接受到的实时数据划分到不同的batch中，然后由Spark Engine处理并生成结果batch

## DStream

Spark Streaming提供一个高级抽象模型——Discretized Stream，DStream，它代表一个持续的数据流

可以对DStream进行一些高级操作，如：map、reduce、join、window等，但是有些RDD操作并没有对DStream开发，如果要使用这些API，需要进行Transform Operation操作将DStream转换为RDD：

```scala
val dStreamToRdd = dStream.transform(rdd => {
  ...
  rdd
})
```

## SparkContext

Spark Streaming中的Context是StreamingContext，它是所有功能的主入口，可以通过两种方法创建：

- SparkConf

  ```scala
  val conf = new SparkConf().setAppName(appName).setMaster(master)
  val ssc = new StreamingContext(conf, Seconds(1))
  ```

- SparkContext

  ```scala
  val ss = ...	//existing SparkContext
  val ssc = new Streaming(sc.Second(1))
  ```

## 数据输入

通过StreamingContext创建DStream，DStream可以指定数据输入源(DStream支持许多数据源，如kafka,flume,TCP socketd)

**TCP socketd：**

```scala
// Create a DStream that will connect to hostname:port, like localhost:9999
val lines = ssc.socketTextStream("localhost", 9999)
```

> lines的类型是DStream，代表从数据服务器上接受到的数据流。lines中的每一条记录是一行文本

## 执行

在设置完计算后，它没有马上执行，需要调用下面代码启动处理：

```scala
ssc.start()             // Start the computation
ssc.awaitTermination()  // Wait for the computation to terminate
```



