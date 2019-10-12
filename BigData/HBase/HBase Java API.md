# HBase Java API

HBase 主要客户端接口由 **org.apache.hadoop.hbase.client** 包中的 `HTable` 类提供。创建 HTable 实例是有代价的，每个实例都需要扫描 **.META.** 表，以检查该表是否存在、是否可用，此外还会执行一些其它操作，这些检查和操作非常耗时。

因此，推荐只创建一次 HTable 实例，而且是每个线程创建一个，然后在客户端应用的生命周期内复用这个对象。如果需要使用多个 HTable 实例，可以考虑 HTablePool 类。

## put 方法

