# HBase集群搭建

- 下载解压

  ```
  https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/
  ```

- 修改配置文件

  - hbase-env.sh

    ```sh
    export JAVA_HOME=/export/servers/jdk
    #HBase使用外部zk
    export HBASE_MANAGES_ZK=false
    ```

  - hbase-site.xml

    ```xml
    <configuration>
        <!-- 指定hbase在HDFS上存储的路径 -->
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://node01:9000/hbase</value>
        </property>
        <!-- 指定hbase是分布式的 -->
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
  scp -r /export/server/hbase  node02:/export/server/
  scp -r /export/server/hbase  node03:/export/server/
  ```

- 启动hbase集群

  - 启动zookeeper集群

    ```
    ./zkServer.sh start
    ```

  - 启动hdfs集群

    ```
    start-dfs.sh
    ```

  - 启动hbase

    ```
    start-hbase.sh
    ```

  - 启动多个HMaster，保证集群的可靠性

    ```
    hbase-daemon.sh start master
    ```

- HBase Web UI

  ```
  http://node01:16010
  ```