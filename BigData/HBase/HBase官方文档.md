# Apache HBase 2.1.5

# HBase 配置

HBase 的所有配置文件都在 *conf/* 目录下，需要确保所有节点的配置文件都相同。

## 配置文件

- backup-masters

  默认不存在，它里面保存了备用 Master 所在节点的 host 列表，一行一个 host

- hadoop-metrics2-hbase.properties

  用来连接 Hadoop 的 Metrics2

- hbase-env.cmd 和 hbase-env.sh

  Windows 和 Linux 脚本，用来设置 HBase 工作环境，包括 Jdk 安装位置、Java 选项和其它环境变量。

- hbase-site.xml

  HBase 的主要配置文件，这个文件中的配置会覆盖 HBase 默认配置，默认配置文件在 *docs/hbase-default.xml*，也可以在 HBase Web UI 的 HBase Configuration 查看所有配置。

- log4j.properties

  HBase log4j 配置文件

- regionservers

  保存在 HBase 集群中运行 RegionServer 的节点的 host，一行一个 host。

可以通过 `xmllint` 命令来检测 xml 文件的格式是否正确：

```shell
xmllint -noout filename.xml
```

## 基本要求

**Java**

下表汇总了 HBase 对应的 Java 版本：

| HBase Version | JDK 7                                                       | JDK 8 | JDK 9                                                        | JDK 10                                                       |
| :------------ | :---------------------------------------------------------- | :---- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 2.0           | [Not Supported](http://search-hadoop.com/m/YGbbsPxZ723m3as) | yes   | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) |
| 1.3           | yes                                                         | yes   | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) |
| 1.2           | yes                                                         | yes   | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) |

**ssh**

HBase 需要使用 ssh 命令来进行集群各节点之间的交互，需要确保可以通过 ssh 访问所有的节点，包括 local 节点。

**DNS**

HBase 需要使用 hostname 来表示自己的 IP 地址

**NTP**

集群各节点的时钟需要是同步的，推荐使用 Network Time Protocol (NTP) 服务。

**文件和进程数的限制 (ulimit)**

HBase 需要一次性打开多个文件，可以通过 `ulimit -n` 命令查看服务器的限制数。推荐 ulimit 至少设置为 10000，一般设置为 1024 被倍数 10240。每个 ColumnFamily 至少有一个 StoreFile，如果改 region 处于加载状态，则可能会超过 6 个 StoreFile。打开文件的个数依赖于 ColumnFamily 的个数和 region 的个数。可以通过下面的公式来粗略计算一个 RegionServer 打开的文件数：

```
(每个 ColumnFamily 的 StoreFiles 个数) * (每个 RegionServer 的 regions 个数)
```

另一个需要设置的是用户一次可以运行进程数，可以通过 `ulimit -u` 设置进程数。

### Hadoop

下表展示了每个 HBase 版本支持的 Hadoop:

| HBase-1.2.x      | HBase-1.3.x | HBase-1.5.x | HBase-2.0.x | HBase-2.1.x |      |
| :--------------- | :---------- | :---------- | :---------- | :---------- | ---- |
| Hadoop-2.4.x     | S           | S           | X           | X           | X    |
| Hadoop-2.5.x     | S           | S           | X           | X           | X    |
| Hadoop-2.6.0     | X           | X           | X           | X           | X    |
| Hadoop-2.6.1+    | S           | S           | X           | S           | X    |
| Hadoop-2.7.0     | X           | X           | X           | X           | X    |
| Hadoop-2.7.1+    | S           | S           | S           | S           | S    |
| Hadoop-2.8.[0-1] | X           | X           | X           | X           | X    |
| Hadoop-2.8.2     | NT          | NT          | NT          | NT          | NT   |
| Hadoop-2.8.3+    | NT          | NT          | NT          | S           | S    |
| Hadoop-2.9.0     | X           | X           | X           | X           | X    |
| Hadoop-2.9.1+    | NT          | NT          | NT          | NT          | NT   |
| Hadoop-3.0.x     | X           | X           | X           | X           | X    |
| Hadoop-3.1.0     | X           | X           | X           | X           | X    |

> - "S" = supported
> - "X" = not supported
> - "NT" = Not tested

**df.datanode.max.transfer.threads**

HDFS DataNode 有一个同一时间使用文件数的上限，可以通过 *conf/hdfs-site.xml* 的 `dfs.datanode.max.transfer.threads` 来进行设置，至少设置为 4096。

### Zookeeper

Zookeeper 需要是 3.4.X 版本。

## 默认配置

### HBase 默认配置

**hbase.tmp.dir**

默认值：`${java.io.tmpdir}/hbase-${user.name}`

本地文件系统中的临时目录。

**hbase.rootdir**

默认值：`${hbase.tmp.dir}/hbase`

region 服务共享的目录，HBase 也会持久化 到这个目录。比如，设置为 HDFS 的 /hbase 目录，可以写为 hdfs://namenode.example.org:9000/hbase。

**hbase.cluster.distributed**

默认值：`false`

HBase 集群使用的模式，设置为 `false` 会使用 standalone 模式，使用 standalone 模式，HBase 和 Zookeeper 会运行在同一个 JVM 中。

**hbase.zookeeper.quorum**

默认值：`localhost`

通过逗号分隔的 Zookeeper 服务列表，这个参数值会跟 *hbase.zookeeper.property.clientPort* 参数值组合起来形成 Zookeeper  的连接地址。

**zookeeper.recovery.retry.maxsleeptime**

默认值：`60000`

重试 zookeeper 操作前的最长休眠时间，单位：ms。

**hbase.local.dir**

默认值：`${hbase.tmp.dir}/local/`

本地文件系统中的一个目录，用户本地存储。

**hbase.master.port**

默认值：`1600`

HBase Master 绑定的端口号

**hbase.master.info.port**

默认值：`16010`

HBase Master Web UI 使用的端口号，设置为 -1 表示不使用 Web UI

**hbase.master.info.bindAddress**

默认值：`0.0.0.0`

HBase Master Web UI 绑定的地址

**hbase.master.fileSplitTimeout**

默认值：`600000`

拆分 region 时，在终止尝试前等待的时间

**hbase.regionserver.port**

默认值：`16020`

HBase RegionServer 绑定的端口

**hbase.regionserver.info.port**

默认值：`16030`

HBase RegionServer Web UI 使用的端口，设置为 -1 表示不运行 HBase RegionServer Web UI

**hbase.regionserver.info.bindAddress**

默认值：`0.0.0.0`

HBase RegionServer Web UI 的地址

**hbase.regionserver.handler.count**

默认值：`30`

RegionServers 使用的 RPC 监听个数，这个属性也被 Master 用来统计 master handler 的个数。太多的 handler 会产生反向的效果，设置这个属性值为 CPU 的倍数。如果只读操作比较多，handler 的个数接近 cpu 的个数比较好，可以从 CPU 个数的两倍开始设置。

**hbase.ipc.server.callqueue.handler.factor**

默认值：`0.1`

决定队列个数的因素，设置为 0，所有的 handler 分享一个队列；设置为 1，每个 handler 都会有自己的一个队列。

**hbase.ipc.server.callqueue.read.ratio**

默认值：`0`

把队列划分为读和写队列，指定 0 不对队列进行划分，也就是说读和写请求会被推送到同一个队列集合；小于 0.5，意味着处理读请求的队列少于处理写请求的队列；指定 1，只有 1 个队列用来处理写请求，其它的队列都用来处理读请求。

**hbase.ipc.server.callqueue.scan.ratio**

默认值：`0`

通过这个参数把读队列划分为：small-read 队列和 long-read 队列。指定小于 0.5 的值，意味着 long-read 队列小于 short-read 队列。

**hbase.regionserver.msginterval**

默认值：`3000`

message 从 RegionServer 到 Master 的时间间隔，单位：ms

**hbase.regionserver.logroll.period**

默认值：`3600000`

commit 日志的滚动周期，不管它已将编辑了多少

**hbase.regionserver.global.memstore.size**

默认值：`none`

在新的更新被阻塞并强制刷新前，region 服务的最大内存。默认为堆栈的 40%。当 region 服务的内存达到 *hbase.regionserver.global.memstore.size.lower.limit*，更行会被阻塞并强制刷新。这个属性的值默认为空，为了使用旧的 *hbase.regionserver.global.memstore.upperLimit* 属性。

**hbase.regionserver.global.memstore.size.lower.limit**

默认值：`none`

强制刷新前，region 服务的最大内存。默认 *hbase.regionserver.global.memstore.size* 的 95%。这个属性的值默认为空，为了使用旧的 *hbase.regionserver.global.memstore.lowerLimit* 属性。

**hbase.regionserver.regionSplitLimit**

默认值：`1000`

当 region 达到这个值，region 不会再被分割。这个不是对 region 个数的硬性限制，但是可以作为 region 服务停止分割的一个准则。

**zookeeper.session.timeout**

默认值：`90000`

ZooKeeper session 的超时时间，单位：ms

**zookeeper.znode.parent**

默认值：`/hbase`

HBase 在 zookeeper 中的根 ZNode。

**zookeeper.znode.acl.parent**

默认值：`acl`

权限控制列表的根 ZNode

**hbase.zookeeper.peerport**

默认值：`2888`

ZooKeeper 节点之间进行通信的端口

**hbase.zookeeper.leaderport**

默认值：`3888`

ZooKeeper 进行 leader 选举的端口

**hbase.zookeeper.property.dataDir**

默认值：`${hbase.tmp.dir}/zookeeper`

这个属性来自 ZooKeeper 的配置文件 zoo.cfg，用来保存快照。

**hbase.zookeeper.property.clientPort**

默认值：`2181`

这个属性来自 ZooKeeper 的配置文件 zoo.cfg，client 连接的端口

### hbase-env.sh

用来设置 HBase 的环境变量，可以设置启动 HBase 时的 JVM 参数，比如堆大小和 GC 配置等。也可以设置 HBase 配置，比如日志目录、ssh 选项和 pid 文件存放的目录等。

### log4j.properties

设置 HBase 日志文件的滚动周期和日志输出级别。修改这个文件后，需要重启 HBase，可以通过 HBase UI 修改特定程序的日志级别。

## HBase Client 

如果通过 IDE 来运行 HBase Client，应该包含 *conf/*  目录到 classpath，这样 *hbase-site.xml* 就可以被读取到。

通过 Maven 构建的 Java 应用程序，可以通过添加下面的依赖来构建 Client:

```xml
<dependency>
  <groupId>org.apache.hbase</groupId>
  <artifactId>hbase-shaded-client</artifactId>
  <version>2.0.0</version>
</dependency>
```

hbase-site.xml 文件的示例：

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>example1,example2,example3</value>
    <description>The directory shared by region servers.
    </description>
  </property>
</configuration>
```

### Java Client

Java Client 使用的配置保存在 **HBaseConfiguration** 实例中，可以通过 `HBaseConfiguration.create()` 方法来创建，在启动的时候会读取 Client 的 Classpath 中的 *hbase-site.xml* 文件，启动时候也会去查找 *hbase-default.xml* 文件，比如在 hbase.X.X.X.jar 中的 *hbase-default.xml*。也可以在程序中指定 ZooKeeper 的地址：

```java
Configuration config = HBaseConfiguration.create();
config.set("hbase.zookeeper.quorum", "localhost");
```

> 如果 ZooKeeper 是一个集群，可以通过逗号分隔来指定 ZK 集群

## 配置示例

### hbase-site.xml

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>example1,example2,example3</value>
    <description>The directory shared by RegionServers.
    </description>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/export/zookeeper</value>
    <description>Property from ZooKeeper config zoo.cfg.
    The directory where the snapshot is stored.
    </description>
  </property>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://example0:8020/hbase</value>
    <description>The directory shared by RegionServers.
    </description>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
    <description>The mode the cluster will be in. Possible values are
      false: standalone and pseudo-distributed setups with managed ZooKeeper
      true: fully-distributed with unmanaged ZooKeeper Quorum (see hbase-env.sh)
    </description>
  </property>
</configuration>
```

### regionservers

```
example1
example2
example3
```

### hbase-env.sh

```shell
# The java implementation to use.
export JAVA_HOME=/usr/java/jdk1.8.0/

# The maximum amount of heap to use. Default is left to JVM default.
export HBASE_HEAPSIZE=4G
```

## 重要的配置

### 必要设置

如果集群中有许多 region，当  Master 启动的时候，其中一个 Regionserver 会先登记，其它的 RegionServers 滞后。第一登记的服务会被分配到所有的 region，但是第一个服务可能不是最佳的。为了阻止这种情况发生，可以设置 `hbase.master.wait.on.regionservers.mintostart` 属性为 1。

### 推荐设置

- ZooKeeper 设置

  **zookeeper.session.timeout**

  默认时长为 3 分钟，这意味着如果 一个服务挂掉，在 Master 注意到服务挂掉，并恢复启动前需要 3 分钟。需要把 timeout 降到 1 分钟甚至更低。在修改这个配置之前，确保 JVM GC 是可控的，否则，如果 GC 的时长超过了 timeout，可能会去掉 RegionServer。

  我们会把这个值设置的高一些，为了防止在大量导入时 RegionServer 挂掉 ，这通常是因为 JVM 没有调整，并且正在运行很长时间的 GC 暂停。

- HDFS 设置

  **dfs.datanode.failed.volumes.tolerated**

  在 DataNode 停止服务前允许失败的卷的数量，默认值为 0，也就是任何卷失败都会导致 datanode 停止服务。这个值可以设置为可选磁盘的一半。

  **hbase.regionserver.handler.count**

  这个配置定义了一直开启的用来回应用户请求的线程数。经验法则是，当每个请求的负载比较大时(big put, 使用大量内存进行扫描)，可以设置的低一些；当请求的负载比较小时(gets, small puts, ICV, 删除)，可以设置的高一些。总的查询个数是通过 **hbase.ipc.server.max.callqueue.size** 进行设置的。

  如果客户端的请求负载比较低，可以设置为最大值，典型的示例是一个用于 web 服务的集群，查询操作比写入操作要更多。但是这个值设置太高的话，发生在一台 region 服务上的 put 操作会给内存带来太大的压力，甚至造成内存溢出。一个运行在低内存上的 RegionServer 可能回到值频繁的 GC，直到 GC 暂停变得非常明显 (原因是不管 GC 如何尝试，所有的内存都用来保持请求的负载不崩溃)。一段时间后，整个集群的吞吐量都会受到影响，因为所有请求 RegionServer 都会花费很长的时间来处理，这个会使问题变得更加严重。

  可以通过每个 RegionServer 上的日志来查看 handler 的所少，可以参考 [rpc.logging](https://hbase.apache.org/2.1/book.html#rpc.logging)

- 压缩

  可以开启 ColumnFamily 压缩，有几种方式是几乎无损耗的，通过减少 StoreFile 的大小来减少 I/O，从而提高性能。可以查看 [compression](https://hbase.apache.org/2.1/book.html#compression) 了解更多信息。

- 设置 WAL 文件的大小和数量

  HBase 通过 WAL 来恢复没有及时刷新到磁盘的内存数据。WAL 文件的大小应该小于 HDFS 块的大小 (默认情况下，HDFS 块大小为 64 MB，WAL 文件大概是 60 MB)。

  HBase 也限制了 WAL 文件的数量，为了确保恢复过程中不需要去重置太多的数据，这个需要根据内存大小进行设置。比如，有 16 GB 的 Region Service 堆内存，默认内存大小设置为 0.4，WAL 文件默认大小为 60 MB，WAL 文件的个数大概是 109 个。然而内存并不是在所有时间都会占满，WAL 文件数也会更少。

# 数据模型

在 HBase 中，数据存储在 table 中，table 中包含行和列。这跟关系型数据库 (RDBMS) 中的属于有些重叠，但是它们并没有什么关联，可以把 HBase 的 table 想成一个多维的 map。

HBase 数据模型属于：

- Table

  table 由多个 Row 组成

- Row

  Row 包含 row key 和 一个或多个 column，row 在存储过程中会根据 row key 的字母顺序进行存储，因此 row key 的设计十分重要，row key 的设计目的是把相互关联的 row 存储在一块。一个常见的 row key 设计是使用网站的域名，并把域名前后颠倒存储(org.apache.www, org.apache.mial, org.apache.jira)，这样所有 apache 的域名都会存储在一起，而不是根据子域名的首字母进行分散存储。

- Column

  Column 包含一个 columnt family 和 column 修饰符，通过 `:` 分隔

- Column Family (列族)

  处于性能考虑，Column Family 实际上是将一组列和其值并置在一起。每个 cloumn family 都有一系列的属性，比如，它的值是否应该缓存在内存中，数据如何进行压缩，row key 是否进行序列化等。每行都有相同的 column families，尽管有些 row 可能不会在 column family 中存储任何数据。

- Column Qualifier

  列修饰符是添加到 column family 来给数据添加索引，给定一个 columnt family *content*，一个列修饰符可能是 *content:html* ，另一个列修饰符可能是 *content:pdf*。column family 在建表时已经固定下来，列修饰符是可变的，并且在 row  之间也可以是不同的。

- Cell

  Cell 由 row、 columnt family、 column qualifier、 值和时间戳组成，时间戳代表值的版本。

- Timestamp

  每个值都会带有一个时间戳，用来标识值的版本。时间戳代表数据写入时 RegionServer 的时间，也可以在写入数据时指定不同的时间戳。

## 概念视图

通过一个示例来说明 HBase 的表结构：

有一张表叫 **webtable**, 它包含两行：**com.cnn.www** 和 **com.example.www**, 三个列族：**contents**、**anchor** 和 **people**。第一行 (com.cnn.www) ，anchor 列族包含两列：**anchor:cssnsi.com** 和 **anchor:my.look.ca**, contents 列族包含一列：**contents:html**。con.cnn.www row-key 有 5 个版本，com.example.www row-key 有 1 个版本。

webtable

| Row Key           | Time Stamp | ColumnFamily `contents`   | ColumnFamily `anchor`         | ColumnFamily `people`      |
| :---------------- | :--------- | :------------------------ | :---------------------------- | :------------------------- |
| "com.cnn.www"     | t9         |                           | anchor:cnnsi.com = "CNN"      |                            |
| "com.cnn.www"     | t8         |                           | anchor:my.look.ca = "CNN.com" |                            |
| "com.cnn.www"     | t6         | contents:html = "<html>…" |                               |                            |
| "com.cnn.www"     | t5         | contents:html = "<html>…" |                               |                            |
| "com.cnn.www"     | t3         | contents:html = "<html>…" |                               |                            |
| "com.example.www" | t5         | contents:html = "<html>…" |                               | people:author = "John Doe" |

> Column 名字是由 prefix 和 qualifier 组成，比如：contents:html

表中空的 cell 并不会占用空间，实际上并不存在。以表格的形式来展示 HBase 的数据有些不准确，一下以多维 map 的格式来展示相同的数据：

```json
{
  "com.cnn.www": {
    contents: {
      t6: contents:html: "<html>..."
      t5: contents:html: "<html>..."
      t3: contents:html: "<html>..."
    }
    anchor: {
      t9: anchor:cnnsi.com = "CNN"
      t8: anchor:my.look.ca = "CNN.com"
    }
    people: {}
  }
  "com.example.www": {
    contents: {
      t5: contents:html: "<html>..."
    }
    anchor: {}
    people: {
      t5: people:author: "John Doe"
    }
  }
}
```

## 物理视图

HBase 是通过 column family 来进行存储的，一个新的 column 标识可以在任何时候添加到已经存在的列族中。

ColumnFamily **anchor**

| Row Key       | Time Stamp | Column Family `anchor`        |
| :------------ | :--------- | :---------------------------- |
| "com.cnn.www" | t9         | anchor:cnnsi.com = "CNN"      |
| "com.cnn.www" | t8         | anchor:my.look.ca = "CNN.com" |

ColumnFamily **contents**

| Row Key       | Time Stamp | ColumnFamily `contents:`  |
| :------------ | :--------- | :------------------------ |
| "com.cnn.www" | t6         | contents:html = "<html>…" |
| "com.cnn.www" | t5         | contents:html = "<html>…" |
| "com.cnn.www" | t3         | contents:html = "<html>…" |

概念视图中展示的空 cell 没有保存，如果请求 contents:html 列的 t8 版本的值会返回 no，请求 anchor:my.look.ca 列的 t9 版本的值也会返回 no。如果没有指定版本，会返回指定列的最新的值。如果指定多个版本，也会返回最新的版本，因为是根据时间戳的降序排列的。

## Namespace

命名空间是表的逻辑分区，这种抽象为下面讲的多用户功能奠定了基础：

- Quota Management，限定命名空间可以消耗的资源 (比如：regions, tables)
- Namespace Security Administration，提供另一个级别的安全管理
- Region server group，把 namespace 或表绑定在 RegionServer 的子集上，保证粗粒度的隔离

**Namespace management**

命名空间可以被创建、移除和修改，命名空间实在创建表的时候通过指定表名来决定的：

```
<table namespace>:<table qualifier>
```

示例：

```
# Create a namespace
create_namespace 'my_ns'

# Create my_table in my_ns namespace
create 'my_na:my_table', 'fam'

# drop  namespace
drop_namespace 'my_ns'

# alter namespace
alter_namespace 'my_ns', {METHOD => 'set', 'PROPERTY_NAME' => 'PROPERTY_VALUE'}
```

**Predefined namespace**

有两种预定义命名空间:

- hbase，系统命名空间，用来包含 HBase 本身的表
- default, 没有明确指定命名空间的表会放在这个命名空间中

示例：

```
# namespace=foo, table qualifier=bar
create 'foo:bar', 'fam'

# namespace=default, table qualifier=bar
create 'bar', 'fam'
```

