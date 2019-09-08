# Spark

Apache Spark 是一个分布式计算系统，它支持多种 API：Java, Scala, Python 和 R，并且包含多种工具：通过 sql 处理结构化数据的 **Spark SQL**, 用于机器学习的 MLlib, 用于图计算的 GraphX 和 Spark Streaming。

Spark 运行在 Java8+, Python 2.7+/3.4+, R 3.1+，对于 Scala API，Spark 2.2.0 使用的是 Scala 2.11，我们可以使用 2.11.X。

在 Spark 2.2.0 版本中，移除了对 java 7，python 2.6 和 Hadoop 小于 2.6.5 版本的支持。

# 提交应用程序

*SPARK_HOME/bin/spark-submit* 脚本是用来提交应用程序到集群。可以通过 sbt 或 Maven 把应用程序打成 jar 包，jar 中 Spark 和 Hadoop 相关的依赖可以设置为 `provided` 级别，因为在运行程序时，集群管理已经提供了这些依赖。

## spark-submit 脚本

命令格式：

```shell
./bin/spark-submit \
 --class <main-class> \
 --master <master-url> \
 --deploy-mode <deploy-mode> \
 --conf <key>=<value> \
 ... # other options
 <application-jar> \
 [application-arguments]
```

> 常用选项说明：
>
> - --class: 应用程序的全路径 (比如，org.apache.spark.examples.SparkPi)
> - --master: 指定集群的 master URL (比如：spark://23.195.26.187:7077)
> - --deploy-mode: cluster, 部署 driver 端到 worker 节点上；client (默认选项), 本地作为一个外部客户端。
> - --conf: 通过 k-v 形式，指定 Spark 配置属性，如果属性值中包含空格，可以通过双引号包含，比如：“key=value"。
> - application-jar: 包含应用程序和依赖度的 jar 包路径，指定这个 URL 必须是集群可以访问的路径。比如，可以指定一个 hdfs:// 或 file:// 路径，这个路径是在所有节点都可以访问的。
> - 传递给主类 main 方法的参数。

一个常用的部署策略是通过一个跟 work 节点协同工作的机器来提交应用程序(比如，一个 standalone 模式集群的 Master 节点)。这种方式非常适合使用 *client* 模式，driver 程序直接在 spark-submit 进程中启动，该进程充当集群的客户端，应用程序的输入和输出都会附加到控制台。因此这个模式非常适合涉及到交互的应用程序。

如果应用程序从一台远离 worker 节点的机器提交(比如，本地机器)，通常使用 *cluster* 模式来最小化 driver 端和 executor 之间的网络延迟。目前，*standalone* 模式不支持 Python 应用程序的 *cluster* 模式。

对于 Python 应用程序，只需要把 \<application-jar> 参数的 jar 包换成 .py 文件，并通过 --py-files 参数把 .zip/.egg/.py 文件添加到搜索路径。

对于一些特定的集群管理模式，还有一些特定的选项可以使用。比如，在 Spark standalone 集群中使用 *cluster* 部署模式，可以指定 *--supervise* 参数来确保 driver 端失败后重启。可以通过 *spark-submit --help* 命令来查看具体的选项。一下是一些示例：

```shell
# 在本地运行，分配 8 核
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[8] \
  /path/to/examples.jar \
  100

# 在 Spark standlone 集群以 client 部署模式运行
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# 在 Spark standlone 集群以 cluster 部署模式运行，并带有 supervise 选项
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --deploy-mode cluster \
  --supervise \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# 在 YARN 集群运行
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode cluster \  # can be client for client mode
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000

# 在 Spark standalone 集群运行 Python 程序
./bin/spark-submit \
  --master spark://207.184.161.138:7077 \
  examples/src/main/python/pi.py \
  1000

# 在 Mesos 集群以 cluster 部署模式运行，并带有 supervise 选项
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master mesos://207.184.161.138:7077 \
  --deploy-mode cluster \
  --supervise \
  --executor-memory 20G \
  --total-executor-cores 100 \
  http://path/to/examples.jar \
  1000
```

## Master URLs

传递给 Spark 的 master URL 可以是一下格式：

| Master URL                      | 含义                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| local                           | 在本地以一个 worker 线程运行 Spark (根本没有并行性)          |
| local[K]                        | 在本地以 K 个 worker 线程运行 Spark (理论上，设置为机器的核数) |
| local[K,F]                      | 在本地以 K 个 worker 线程运行 Spark，并设置 task 最大失败次数为 F |
| local[*]                        | 在本地运行 Spark，worker 的线程数跟机器的洛基核个数一致      |
| local[*,F]                      | 在本地运行 Spark，worker 的线程数跟机器的洛基核个数一致，并设置 task 最大失败次数为 F |
| spark://HOST:PORT               | 连接到给定的 Spark standalone 集群的 master，这个端口必须是 master 配置使用的端口，默认为 7077 |
| spark://HOST1:PORT1,HOST2:PORT2 | 连接到通过 Zookeeper 配置了主备的 Spark standalone 集群。这个列表中必须包括高可用中配置的所有 master 的主机名。端口默认为 7077 |
| mesos://HOST:PORT               | 连接到 Mesos 集群。端口默认为 5050                           |
| yarn                            | 通过 *client* 或 *cluster* 部署模式连接到 yarn 集群。集群的位置会通过 *HADOOP_CONF-DIR* 或 *YARN_CONF_DIR* 变量来找到 |

## 从文件中加载配置

*spark-submit* 脚本可以从配置文件加载默认 Spark 配置到应用程序。默认情况下，会从 *conf/spark-default.conf* 文件读取配置。

使用默认的 Spark 配置，可以在使用 spark-submit 脚本时省略一些选项，比如，如果配置文件中设置了 *spark.master*  属性，可以在使用 spark-submit 脚本是省略 *--master* 选项。通常情况下，通过 SparkConf 的配置有最好的优先级，然后是在 *spark-submit* 脚本中指定的，最后是默认配置文件中的。

如果你不清楚配置项是从哪里来的，可以在运行 *spark-submit* 脚本是加上 *--verbose* 选项，来输出 debug 信息。

## 依赖管理

当使用 spark-submit 脚本，应用程序的 jar 包依赖的任何 jar 包，都可以通过 *--jars* 选项来上传到集群，如果依赖的是多个 jar 包，可以通过逗号分隔。jar 包列表会被包含在 driver 和 excutor 类路径中。

在 Spark 中传递 jar 包，可以使用以下的 URL 方案：

- file: -绝对路径 和 file:/ URIs，由 driver 的 HTTP 文件服务提供，并且每个 executor 从 driver 的 HTTP 服务拉取文件。
- hdfs:, http:, https:, ftp: 这些是从 URI 中拉取文件和 jar 包
- local: 一个以 local: 开始的 URI, 以本地文件的形式存在于每个 worker 节点上。这意味着不会有网络 IO，大的文件或 jar 包，会被发送到每个 worker 节点，或通过 NFS，GlusterFS 进行分享

jar 包和文件会被复制到 executor 节点的每个 SparkContext 的工作目录。随着时间推移，这个会占用许多空间，并且需要并清理。如果使用 YARN，会自动清理，如果使用的 Spark standalone，可以通过配置 *spark.worker.cleanup.appDataTtl* 属性来设置自动清理。

用户可以通过 *--packages* 指定 Maven 坐标列表(以逗号分隔)来引入其他依赖。使用这个命令时，会处理所有的依赖项。