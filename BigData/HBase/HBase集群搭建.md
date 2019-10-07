# HBase集群搭建

## 安装 JDK

安装 JDK，HBase 2.0 需要安装 JDK 8。

## 安装 HBase

- 下载解压

  下载地址：http://archive.apache.org/dist/hbase/ 或 https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/

- 解压

  ```shell
  $ tar zxvf hbase-2.0.0-bin.tar.gz
  ```

- 修改配置文件

  - hbase-env.sh

    ```sh
    export JAVA_HOME=/export/servers/jdk
    # 是否使用 HBase 内置的 Zookeeper
    export HBASE_MANAGES_ZK=false
    ```

  - hbase-site.xml

    ```xml
    <configuration>
        <!-- hbase 数据存储的路径 -->
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://node01:9000/hbase</value>
        </property>
        <!-- hbase 是否为集群部署 -->
        <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
        <!--  zookeeper 的地址 -->
        <property>
            <name>hbase.zookeeper.quorum</name>
            <value>node01,node02,node03</value>
        </property>
        
        <!-- 使用内置 zookeeper 时, 保存 zk 数据的目录 -->
        <property>
      		<name>hbase.zookeeper.property.dataDir</name>
      		<value>/usr/local/zookeeper</value>
    	</property>
    </configuration>
    ```

  - regionservers

    ```
    node02
    node03
    ```

  - backup-masters，指定备用的主节点

    ```
    node02
    ```

  修改完配置文件后需要把这些配置文件拷贝到集群的其它节点，可以拷贝 *conf/* 整个目录

- 拷贝hbase到其它节点

  ```shell
  $ scp -r /export/server/hbase  node02:/export/server/
  $ scp -r /export/server/hbase  node03:/export/server/
  ```

- 启动hbase集群

  - 启动zookeeper集群

    ```shell
    $ ZK_HOME/bin/zkServer.sh start
    ```

  - 启动hdfs集群

    ```shell
    $ HADOOP_HOME/sbin/start-dfs.sh
    ```

  - 启动hbase

    ```shell
    $ ./bin/start-hbase.sh
    ```

  - 启动多个HMaster，保证集群的可靠性

    ```shell
    $ ./bin/hbase-daemon.sh start master
    ```
    
    > 通过 `jps` 命令会看到 `HMaster` 和 `HRegionServer`，如果使用的内置 zookeeper, 还会看到 `HquorumPeer`

- 停止 HBase 集群

  ```shell
  $ ./bin/stop-hbase.sh
  ```
  
- HBase Web UI

  Master:

  ```
  http://node01:16010
  ```

  RegionServer:

  ```
  http://node01:16030
  ```

- HBase Shell

  ```shell
  $ ./bin/hbase shell
  ```


# HBase 配置



## 配置文件简介

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

- java

  | HBase Version | JDK 7                                                       | JDK 8 | JDK 9                                                        | JDK 10                                                       |
  | :------------ | :---------------------------------------------------------- | :---- | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | 2.0           | [Not Supported](http://search-hadoop.com/m/YGbbsPxZ723m3as) | yes   | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) |
  | 1.3           | yes                                                         | yes   | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) |
  | 1.2           | yes                                                         | yes   | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) | [Not Supported](https://issues.apache.org/jira/browse/HBASE-20264) |

- ssh

- DNS

- NTP

  所有节点的时钟必须是同步的

- 用户打开文件个数

  可以通过 `ulimit -n` 命令查看，这个数值要大于 10000，一般会设置为 10240 这样的数值。

  每个 ColumnFamily 至少有一个 StoreFile，如果改 region 处于加载状态，则可能会超过 6 个 StoreFile。打开文件的个数依赖于 ColumnFamily 的个数和 region 的个数。可以通过下面的公式来粗略计算一个 RegionServer 打开的文件数：

  ```
  (每个 ColumnFamily 的 StoreFiles 个数) * (每个 RegionServer 的 regions 个数)
  ```

## Hadoop 设置

**Hadoop 版本**

"S" = supported

"X" = not supported

"NT" = Not tested

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

**df.datanode.max.transfer.threads**

HDFS DataNode 有一个同一时间使用文件数的上限，可以通过 *conf/hdfs-site.xml* 的 `dfs.datanode.max.transfer.threads` 来进行设置，至少设置为 4096。

## Zookeeper 版本

Zookeeper 需要是 3.4.X 版本。