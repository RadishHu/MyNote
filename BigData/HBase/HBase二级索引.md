# HBase 二级索引

HBase 里只有 rowkey 作为一级索引，如果使用非 rowkey 字段进行数据查询，需要通过 MapReduce/Spark 等分布式计算框架进行，时间延迟会比较高。因此需要在 HBase 上构建二级索引，来使用非 rowkey 字段进行快速查询。

# 二级索引方案

## 基于 Coprocessor 方案

从 0.94 版本开始，HBase 官方文档提出 HBase 实现二级索引的一种方法：

- 基于 Coprocessor，开发自定义数据处理逻辑，采用数据 "双写" (dual-write) 策略，在有数据写入时同步到二级索引表



# 参考

[HBase二级索引方案](https://zhuanlan.zhihu.com/p/43972378)

