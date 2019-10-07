# HBase

# HBase 简介

[BigTable 的开源实现：HBase](https://time.geekbang.org/column/article/70253)

谷歌发表 GFS、MapReduce、BigTable 三篇论文，号称 ”三架马车“，GFS 对应 Hadoop 的 HDFS，MapReduce 对应 Hadoop 的 MapReduce，BigTable 对应 NoSQL 系统 HBase。

关系型数据库和非关系型数据库的区别：

关系型数据库 (RDBMS)，在传统企业的应用领域，许多应用都是面向数据库设计，也就是先设计数据库然后设计程序。但是关系型数据库在面对含量数据处理时。



[可能是最易懂的 HBase 架构原理解析](https://mp.weixin.qq.com/s/cO7MrAXjat9Gg_BHWcVEpw)

HDFS 的缺点：不支持小文件，不支持并发写，不支持文件随机读写，查询效率低。

HBase 是列式存储，MySQL 是行式存储。

[BigTable 的开源实现：HBase](https://time.geekbang.org/column/article/70253)

## HBase 可伸缩架构

HBase 的伸缩性主要依赖其可分裂的 HRegion 及可伸缩的分布式文件系统 HDFS 实现。

![](https://static001.geekbang.org/resource/image/9f/f7/9f4220274ef0a6bcf253e8d012a6d4f7.png)

HRegion 是 HBase 负责数据存储的主要进程，应用程序对数据的读写操作都是通过和 HRegion 通信完成的。

HRegionServer 是物理服务器，每个 HRegionServer 上可以启动多个 HRegion 实例，当一个 HRegion 写入的数据太多，达到配置的阈值，一个 HRegion 会分裂成两个 HRegion，并将 HRegion 在整个集群中进行迁移，以使 HRegionServer 的负载均衡。

每个 HRegion 存储地段 Key 值区间 [key1, key2) 的数据，所有 HRegion 的信息，包括存储的 Key 值区间，所在 HRegionServer 地址、访问端口号等，都记录在 HMaster 服务器上。为了保证 HMaster 的高可用，HBase 会启动多个 HMaster，并通过 ZooKeeper 选举出一个主服务器。

应用程序读写数据的流程如下图所示：

![](https://static001.geekbang.org/resource/image/9f/ab/9fd982205b06ecd43053202da2ae08ab.png)

应用程序通过 ZooKeeper 获取主 HMaster 的地址，输入 Key 值获得这个 Key 所在 HRegionServer 地址，然后请求 HRegionServer 上的 HRegion，获得所需要的数据。

数据写入流程也一样，HRegion 会把数据存储在若干个 HFile 格式的文件中，这些文件使用 HDFS 存储，保证数据高可用。当一个 HRegion 中数据太多时，这个 HRegion 连同 HFile 会分裂成两个 HRegion，并根据集群中服务器负载进行迁移。如果集群中有新加入的服务器，即有新的 HRegionServer，由于它的负载比较低，会把 HRegion 迁移获取并记录到 HMaster。

## HBase 可扩展数据模型

传统的关系型数据库为了保证关系运算 (通过 SQL 语句) 的正确性，在设计表结构的时候，需要指定表的 schema，即表的字段名称、数据类型，并要遵循特定的设计范式。这种僵硬的数据结构难以面对需求变更带来的挑战，有些设计者会通过预先设计一些冗余字段来应对。

许多 NoSQL 数据库使用列族 (ColumnFamily) 来解决表结构设计的扩展性问题。使用列族，在创建表时，只需要指定列族的名字，无需指定字段 (Column)，在写入数据时再指定字段名，这样就可以随意扩展应用程序的数据结构。

## HBase 的高性能存储

传统的机械式磁盘的访问特性是**连续读写很快，随机读写很慢**，因为机械磁盘靠电机驱动访问磁盘上的数据，电机要将磁头落到数据所在的磁道上，这个过程需要较长的寻址时间。如果数据不连续存储，磁头就要不停的移动，浪费大量的时间。

为了提高数据写入速度，HBase 使用 **LSM 树**的数据结构进行数据存储，LSM 树全程 Log Structed Merge Tree，也就是 Log 结构合并树。数据写入时以 Log 方式连续写入，然后异步对磁盘上的多个 LSM 树进行合并。

![](https://static001.geekbang.org/resource/image/5f/3b/5fbd17a9c0b9f1a10347a4473d00ad3b.jpg)

LSM 树可以看作是一个 N 阶合并树，数据写操作 (包括插入、修改、删除) 都在内存中进行，并且都会创建一个新记录 (修改会创建新的数据值，删除会记录一个删除标志)。这些数据在内存中仍然还是一颗排序树，当数据量超过设定的内存阈值后，会将这课排序树和磁盘上最新的排序树合并。当这棵树的数据量也超过阈值后，会和磁盘上下一级的排序树合并。合并过程中，会用新更新的数据覆盖旧的数据 (或记录为不同版本)。

在需要进行读操作时，会从内存中的排序树开始搜索，如果没有找到，就从磁盘上的排序树顺序查找。在 LSM 树上进行一次数据更新不需要访问磁盘，在内存即可完成。当数据以写操作为主，而读操作则集中在最近写入数据时，使用 LSM 树可以极大地减少磁盘的访问次数，加快访问速度。

> 通过 LSM 树的存储方式，是数据可以通过连续写磁盘的方式保存数据，极大地提高了数据写入性能。

