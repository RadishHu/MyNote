# 简介

Flume 是一个分布式的用于高效采集、聚合和移动大量日志数据的服务，它可以从不同的数据源采集数据到一个数据存储中心。它有一个简单和灵活的基于数据流的框架。

Flume 的是使用不仅受限于日志采集。因为数据源是可以自定义的，Flume 可以用来移动大量的数据包括：网络通信数据，社交媒体形成的数据，邮件信息等。

## 系统要求

1. Java 1.8 或更版本
2. 足够的内存，用于配置 sources, channels 或 sinks
3. 足够的自盘空间，用于配置 channels 或 sinks
4. 目录的读写权限

# 架构

## 数据流模型

`event` 是 Flume 数据流的基本单元，它包含一个字节的有效负载和一个可选的属性集合。

`agent` 是一个 JVM 进程，它包含 `source`, `channel`, `sink` 等组件，它利用这些组件将 `event` 从数据源传送到目的地(或另一个 `agent`节点)。

`source` 消费从外部的数据源(或另外一个 agent 节点的 sink)传送过来的 `event`，然后将 `event` 存储到 `channel` 中，最后 `event` 被 `sink` 传送到目标数据源(或另外一个 agent 节点的 source)。

## 可靠性

Flume 使用事务来保证 event 传递的可靠性。source 将 event 存储在 channel 中 和 sink 从 channel 中去除 event 分别放在两个事务中。在多节点的数据流中，event 从上一个节点的 sink 到 下一个节点的 sink 同样通过事务来保证数据传递的可靠性。

## 可恢复性

Flume 可以通过 file channel 持久化数据到本地文件系统，这样 event 数据在丢失后可以从 channel 中恢复。而 memory channel 把数据保存在一个内存队列中，这样虽然 event 传递比较快，但是当 agent 挂掉后，channel 中的数据是无法恢复的。

# 创建

## 创建 agent

Flume agent 的在本地的一个配置文件中进行配置，这个配置文件是一个文本文件，格式跟 Java properties 文件一样。可以在一个配置文件中配置多个 agent，配置文件中包括 agent  的每个 source, sink, channel 的属性，以及它们是如何组装在一起来形成数据流。

### 配置组件

一个agent 由三部分组成: source, sink 和 channle， 每个组件都有一个名字、类型和一系列的属性。比如，一个 Avro 类型的 source 需要数据源的 hostname 和一个端口，一个内存 channel 可以指定队列的大小，HDFS sink 需要指定文件系统的 URI、文件存储的路径和频率。这些属性都需要在配置文件中去配置。

### 连接组件

为了组成一个数据流，agent 需要知道每个组件之间是如何连接的。flume 通过列出 agent 中每一个 source、sink 和 channle 的名字，然后指定 channel 跟 sink、source 的连接关系。

### 启动 agent

agent 通过 FLUME_HOME/bin 下的 `flume-ng` shell 脚本启动，启动命令中需要指定 agent 名字、config 目录和配置文件。

```shell
flume-ng agent -n [agent_name] -c conf -f conf/flume-conf.properties
```

### 示例

配置文件 example.conf：

```properties
# example.conf: A single-node Flume configuration
# agent 中各个组件的名字
a1.sources = r1
a1.sinks = k1
a1.channels = c1
# 配置 source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444
# 配置 sink
a1.sinks.k1.type = logger
# 配置 channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
# 关联 source, sink 和 channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

> 这是一个单节点 agent 的配置文件，agent 名字为 a1
>
> a1 的 source 名为 r1，r1 监听 44444 端口
>
> a1 的 channel 名为 c1，缓存 event 数据在内存中
>
> a1 的sink 名为 k1，把 event 数据输出到控制台

启动 agent:

```shell
flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console
```

> --conf=\<conf-dir\>，\<conf-dir\> 目录中包含一个 shell 脚本 flume-env.sh 和一个 log4j.properties 配置文件

在另一个终端中 telnet 端口 44444 并往 flume 发送数据：

```
$ telnet localhost 44444
```

### 在配置文件中使用环境变量

Flume 可以在配置文件中使用环境变量，比如：

```properties
a1.sources = r1
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = ${NC_PORT}
a1.sources.r1.channels = c1
```

> 环境变量只能在 value 中使用，不要可以在 key 中使用

在设置 `propertiesImplementation = org.apache.flume.node.EnvVarResolverProperties` 后，可以在启动 agent 时，通过 java 属性来传递变量值，比如：

```shell
$ NC_PORT=44444 bin/flume-ng agent –conf conf –conf-file example.conf –name a1 -Dflume.root.logger=INFO,console -DpropertiesImplementation=org.apache.flume.node.EnvVarResolverProperties
```

> 这只是一个示例，还可以通过其它方法来设置环境变量，比如在 flume-env.sh 文件中配置

记录原始数据

在生产环境日志中输出原始数据流不是一个明智之举，这样会在 flume 日志中泄露一些敏感的信息。flume 不会在日志中输出这样的信息。

如果想要在日志中输出 event 或配置相关的数据，可以配置 log4j 的属性和一些 java 属性：

- 输出配置相关的数据，可以设置 java 的 `-Dorg.apache.flume.log.printconfig=true` 属性，这个可以在启动命令里设置，也设置 flume-env.sh 文件中的 `JAVA_OPTS` 变量。
- 输出 event 数据，可以通过启动命令设置 java 的 `-Dorg.apache.flume.log.rawdata=true` 属性，同时还必须设置 log4j 的日志输出级别为 `DEBUG` 或 `TRACE` 来使 event 数据输出到 flume 日志中。

示例：

```shell
$ bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=DEBUG,console -Dorg.apache.flume.log.printconfig=true -Dorg.apache.flume.log.rawdata=true
```

> 通过这个启动命令，可以在 flume 日志中看到 event 和配置相关的信息

### 使用 Zookeeper 保存配置文件

Flume 支持通过 Zookeeper 来配置 agent，这是一个实验性的方法。配置文件需要上传到 Zookeeper，配置文件是保存在 Zookeeper 节点的数据。以下是配置 a1 和 a2 两个 agent 的 Zookeeper 节点树示例：

```
./flume
	|./a1 [Agent config file]
	|./a2 [Agent config file]
```

上传了配置文件后，可以通过这个命令来启动 agent:

```shell
$ bin/flume-ng agent –conf conf -z zkhost:2181,zkhost1:2181 -p /flume –name a1 -Dflume.root.logger=INFO,console
```

> z，指定 Zookeeper 的 host 和端口
>
> p，Agent 配置文件在 Zookeeper 的路径

### 安装第三方插件

