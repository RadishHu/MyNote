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
    # 是否使用 HBase 自己的 Zookeeper
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
        <!-- 指定zk的地址，多个用“,”分割 -->
        <property>
            <name>hbase.zookeeper.quorum</name>
            <value>node01:2181,node02:2181,node03:2181</value>
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
    
    > 通过 `jps` 命令会看到 `HMaster` 和 `HRegionServer`

- 停止 HBase 集群

  ```shell
  $ ./bin/stop-hbase.sh
  ```
  
- HBase Web UI

  ```
  http://node01:16010
  ```

- HBase Shell

  ```shell
  $ ./bin/hbase shell
  ```

  