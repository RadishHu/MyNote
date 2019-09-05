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

![agent](http://flume.apache.org/_images/UserGuide_image00.png)

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

Flume 有一个完整地插件架构，flume 运行着许多外部的 sources, channel, sinks 上，它们的运行都是跟 flume 分开的。

Flume 可以运行自定的组件，需要把自定义组件的 jar 包添加到 flume-env.sh  文件中 FLUME_CLASSPATH 变量指定的目录中。Flume 现在有一个 `plugins.d` 可以自动获取以特定格式打包的插件，这样更容易管理插件。

**plugins.d 目录**

`plugins.d` 目录在 $FLUME_HOME/plugins.d，在启动的时候，`flume-ng` 脚本会在 `plugins.d` 目录下查找合适的组件并把它们的绝对路径包括在启动的 `java` 进程中。

**目录结构**
`plugins.d` 目录中的每个插件都有三个子目录：

- lib - 插件的 jar 包
- libext - 插件依赖的 jar 包
- native - 本地依赖库，比如 `.so` 文件

示例：

```
plugins.d/
plugins.d/custom-source-1/
plugins.d/custom-source-1/lib/my-source.jar
plugins.d/custom-source-1/libext/spring-core-2.5.6.jar
plugins.d/custom-source-2/
plugins.d/custom-source-2/lib/custom.jar
plugins.d/custom-source-2/native/gettext.so
```

## 数据源

Flume 支持多种从外部数据获取数据的方式。

### RPC

一个包含在 flume 中的 Avro 客户端，可以通过 avro RPC 机制发送一个文件到 flume 的 `Avro source` :

```shell
$ bin/flume-ng avro-client -H localhost -p 41414 -F /usr/logs/log.10
```

> 这个命令发送 /usr/logs/log.10 到 flume source 监听的端口

### 执行命令

Flume 中有一个 `exec source` ，它通过执行一个给定的命令，并获取这个命令的输出作为传输数据，命令输出的一个行是一条数据(以 '\r'、'\n'或'\r\n' 结尾)。

### 网络数据源

Flume 支持以下几种机制来获取网络数据：

- Avro
- Thrift
- Syslog
- Netcat

## 创建多级 agent 数据流

![多级 agent](http://flume.apache.org/_images/UserGuide_image03.png)

要在多级 agent 中传递数据，上一级 agent 的 sink 和下一级的 source 需要时 avro 类型的，并且 sink 指向 source 的 hostname 和 port.

## 合并数据流

日志采集中一个非常常见的方案：非常多的生产日志的客户端把日志数据发送到几个跟存储系统关联着的 agent 中。比如，成千上百的 web 服务把发送日志到十几个 agent 中，然后 agent 把日志写入 HDFS。

![](http://flume.apache.org/_images/UserGuide_image02.png)

这个方案通过设置第一层级的 agent 为 `avro sink`，并指向一个单一 agent 的 `avro source` (也可以使用 `thrift source/sink/client`)。第二层级的 source 聚集数据到一个单一的 `channel` 并将数据发送到最终的目的地。

## 多路传输的数据流

Flume 支持传输数据到一个或多个目的地，通过定义一个多路的数据流，可以复制一个 event 到多个 channle 中。

![](http://flume.apache.org/_images/UserGuide_image01.png)

这个示例展示了一个 `source` 发送数据到三个不同的 `channel` 中。发送数据到多个 `channel` 既可以通过复制，也可以通过分散。如果是复制数据，一个 event 会被发送到每个 channel 中。如果是分散发送，一个 event 会被发送到一个跟 event 的属性值相配置的 channel 中，这个可以在 agent 的配置文件中进行配置。

# 配置

Flume agent 的配置是从一个格式类似于 java 配置文件的文件中读取。

## 定义数据流

在一个单一的 agent 中定义数据流，需要通过 channel 来连接 source 和 sink。我们需要列出 agent 的 source、sink 和 channle，然后指定 source 和 sink 到同一个 channel。一个 source 可以连接多个 channel，但是一个 sinke 只能跟一个 channel 连接。配置文件的格式如下：

```properties
# list the sources, sinks and channels for the agent
<Agent>.sources = <Source>
<Agent>.sinks = <Sink>
<Agent>.channels = <Channel1> <Channel2>
# set channel for source
<Agent>.sources.<Source>.channels = <Channel1> <Channel2> ...
# set channel for sink
<Agent>.sinks.<Sink>.channel = <Channel1>
```

示例：

```properties
# list the sources, sinks and channels for the agent
agent_foo.sources = avro-appserver-src-1
agent_foo.sinks = hdfs-sink-1
agent_foo.channels = mem-channel-1
# set channel for source
agent_foo.sources.avro-appserver-src-1.channels = mem-channel-1
# set channel for sink
agent_foo.sinks.hdfs-sink-1.channel = mem-channel-1
```

> 这里的 agent 名为 agent_foo，它从外部的 avro 客户端读取数据，然后通过 memory channel 将数据发送到 HDFS 。

## 设置 agent 中的每个组件

定义了数据流之后，我们需要设置每个 source、sink 和 channle 的属性。这个也是在同一个配置文件找中进行设置，你可以设置每个组件的类型和属性值：

```properties
# properties for sources
<Agent>.sources.<Source>.<someProperty> = <someValue>
# properties for channels
<Agent>.channel.<Channel>.<someProperty> = <someValue>
# properties for sinks
<Agent>.sources.<Sink>.<someProperty> = <someValue>
```

`type` 这个属性对于每个组件来说是必须要设置的，source、sink 和 channel 都有它们自己的一套属性。下面是完善从 avro source 采集数据最后存放到 HDFS 的数据的配置文件：

```properties
agent_foo.sinks = hdfs-Cluster1-sink
agent_foo.channels = mem-channel-1
# set channel for sources, sinks
# properties of avro-AppSrv-source
agent_foo.sources.avro-AppSrv-source.type = avro
agent_foo.sources.avro-AppSrv-source.bind = localhost
agent_foo.sources.avro-AppSrv-source.port = 10000
# properties of mem-channel-1
agent_foo.channels.mem-channel-1.type = memory
agent_foo.channels.mem-channel-1.capacity = 1000
agent_foo.channels.mem-channel-1.transactionCapacity = 100
# properties of hdfs-Cluster1-sink
agent_foo.sinks.hdfs-Cluster1-sink.type = hdfs
agent_foo.sinks.hdfs-Cluster1-sink.hdfs.path = hdfs://namenode/flume/webdata
```

## agent 中添加多个数据流

一个 agent 中可以包含多个单独的数据流，可以在一个配置文件中设置多个 source、sink 和 channel。通过它们可以组合成一个多重的数据流：

```properties
# list the sources, sinks and channels for the agent
<Agent>.sources = <Source1> <Source2>
<Agent>.sinks = <Sink1> <Sink2>
<Agent>.channels = <Channel1> <Channel2>
```

示例：

```properties
# list the sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source1 exec-tail-source2
agent_foo.sinks = hdfs-Cluster1-sink1 avro-forward-sink2
agent_foo.channels = mem-channel-1 file-channel-2
# flow #1 configuration
agent_foo.sources.avro-AppSrv-source1.channels = mem-channel-1
agent_foo.sinks.hdfs-Cluster1-sink1.channel = mem-channel-1
# flow #2 configuration
agent_foo.sources.exec-tail-source2.channels = file-channel-2
agent_foo.sinks.avro-forward-sink2.channel = file-channel-2
```

> 这个示例的 agent 中包含两个数据流：
>
> 1. 从外部的 avro 客户端采集数据保存到 HDFS中
> 2. 通过 tail source 采集数据，输送到 avro sink 中

## 配置由多个 agent 组成的数据流

创建一个多层的数据流，第一级  agent 需要为 avro/thrift sink，并指向下一级 agent 的 avro/thrift source，这样第一级 agent 就可以发送数据到第二级。

示例：
web log agent

```properties
# list sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source
agent_foo.sinks = avro-forward-sink
agent_foo.channels = file-channel
# define the flow
agent_foo.sources.avro-AppSrv-source.channels = file-channel
agent_foo.sinks.avro-forward-sink.channel = file-channel
# avro sink properties
agent_foo.sinks.avro-forward-sink.type = avro
agent_foo.sinks.avro-forward-sink.hostname = 10.1.1.100
agent_foo.sinks.avro-forward-sink.port = 10000
# configure other pieces
#...
```

hdfs agnet

```properties
# list sources, sinks and channels in the agent
agent_foo.sources = avro-collection-source
agent_foo.sinks = hdfs-sink
agent_foo.channels = mem-channel
# define the flow
agent_foo.sources.avro-collection-source.channels = mem-channel
agent_foo.sinks.hdfs-sink.channel = mem-channel
# avro source properties
agent_foo.sources.avro-collection-source.type = avro
agent_foo.sources.avro-collection-source.bind = 10.1.1.100
agent_foo.sources.avro-collection-source.port = 10000
# configure other pieces
#...
```

> 连接 weblog agent 的 avro-sink 到 hdfs agent 的 avro-source，这样就可以把从外部服务获取到的数据存储到 HDfS

## 发散数据流

flume 支持从一个 source 发送数据到多个 channel，有两种模式：

1. 复制，一个 event 被发送到多个 channel 中
2. 分发，一个 event 只会被分发到指定的 channel 中

发散数据流中，需要为一个 source 指定一个 channel 列表，并指定数据分发策略。可以通过 `selector` 属性来指定，这个属性值可以是 `replicating` 或 `multiplexing`，如果没有指定 `selector` 属性，默认为 `replicating`：

```properties
# List the sources, sinks and channels for the agent
<Agent>.sources = <Source1>
<Agent>.sinks = <Sink1> <Sink2>
<Agent>.channels = <Channel1> <Channel2>
# set list of channels for source (separated by space)
<Agent>.sources.<Source1>.channels = <Channel1> <Channel2>
# set channel for sinks
<Agent>.sinks.<Sink1>.channel = <Channel1>
<Agent>.sinks.<Sink2>.channel = <Channel2>
<Agent>.sources.<Source1>.selector.type = replicating
```

`selector` 有一系列的配置属性，其中可以通过设置 mapping 来指定 channel。`selector` 会查看每个 event 指定的 header, event 会被发送到所有与这个 header 值匹配的 channel 中，如果没有匹配的，event 会被发送到默认的 channel 中：

```properties
# Mapping for multiplexing selector
<Agent>.sources.<Source1>.selector.type = multiplexing
<Agent>.sources.<Source1>.selector.header = <someHeader>
<Agent>.sources.<Source1>.selector.mapping.<Value1> = <Channel1>
<Agent>.sources.<Source1>.selector.mapping.<Value2> = <Channel1> <Channel2>
<Agent>.sources.<Source1>.selector.mapping.<Value3> = <Channel2>
#...
<Agent>.sources.<Source1>.selector.default = <Channel2>
```

示例：

```properties
# list the sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source1
agent_foo.sinks = hdfs-Cluster1-sink1 avro-forward-sink2
agent_foo.channels = mem-channel-1 file-channel-2
# set channels for source
agent_foo.sources.avro-AppSrv-source1.channels = mem-channel-1 file-channel-2
# set channel for sinksagent_foo.sinks.hdfs-Cluster1-sink1.channel = mem-channel-1
agent_foo.sinks.avro-forward-sink2.channel = file-channel-2
# channel selector configuration
agent_foo.sources.avro-AppSrv-source1.selector.type = multiplexing
agent_foo.sources.avro-AppSrv-source1.selector.header = State
agent_foo.sources.avro-AppSrv-source1.selector.mapping.CA = mem-channel-1
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.NY = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.default = mem-channel-1
```

> selector 查看名为 "State" 的 header, 如果 header 的值为 "CA", event 被发送到 mem-channel-1 中；如果 header 的值为 "AZ", event 被发送到 file-channel-2 中；如果 header 的值为 "NY", event 被发送到两个 channel 中；如果 header 没有设置值，event 会被发送到 "mem-channel-1" 中。

`selector` 支持 optional channel，给 optional 指定一个 header, 'optional' 参数的使用方法如下：

```properties
# channel selector configuration
agent_foo.sources.avro-AppSrv-source1.selector.type = multiplexing
agent_foo.sources.avro-AppSrv-source1.selector.header = State
agent_foo.sources.avro-AppSrv-source1.selector.mapping.CA = mem-channel-1
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.NY = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.optional.CA = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.default = mem-channel-1
```

selector 首先会尝试把 event 写入 required channel, 如果其中有一个 channel 获取数据失败，会触发事务回滚，然后会重新尝试往所有的 channel 中发送数据。一旦所有的 required channel 都获取到了数据，selector 会尝试把数据发送到 optional channel，只要任何一个 optional channel 获取数据失败，那么这个 event 会被忽略并且不会重试。

如果在 optional channel 和 required channel 中有重复的，那么这个 channel 会被认定为 required, 并且如果一个 channel 失败，会导致整个 required channel 重试。在上面那个示例中，"CA" 对应的 mem-channel-1 虽然既被标记为 required channel 也被标记为 optional channel，但是它会被认为是一个 required channel，往这个 channel 中写入数据失败会导致配置在这个 selector 中的所有 channel 重试。

如果一个 event 的 header 没有对应任何 required channel，event 会被写入 default channel, 并且尝试写入那个 event 对应的 optional channel 中。如果没有指定 required channel 而指定了 optional channel, event 仍然会写入 default channel。如果没有指定 required channel 也没有指定 default channel, 只指定了 optional channel，那所有的发送失败的 event 都不会重试。

## SSL/TLS

有一些 flume 组件支持 SSL/TLS 协议，为了跟其他系统安全地发送数据。

| Component                   | SSL server or client |
| --------------------------- | -------------------- |
| Avro Source                 | server               |
| Avro Sink                   | client               |
| Thrift Source               | server               |
| Thrift Sink                 | client               |
| Kafka Source                | client               |
| Kafka Channel               | client               |
| Kafka Sink                  | client               |
| HTTP Sink                   | server               |
| JMS Source                  | client               |
| Syslog TCP Source           | server               |
| Multiport Syslog TCP Source | server               |

SSL 组件有几个配置项来设置 SSL，不如 SSL flag, 密钥库等。

SSL 的配置是组件级别的，也就是所一些组件可以配置使用 SSL，而其他组件可以配置不使用 SSL。

密钥库可以在组件级别配置，也可以全局配置。

配置组件级别的密钥库，通过在 agent 的配置文件中给相应的组件添加配置参数。这种方式的有点是可以给不同的组件使用不同的密钥库。缺点是每个组件都要配置一下密钥库。组件级别的密钥库是可选的，但是一旦配置了，那么它就有更高的优先级。

配置全局的密钥库，可以值配置一次密钥库，然后给所有的组件使用。全局的可以通过系统数据或环境变量来配置：

| System property                  | Environment variable           | Description                                                  |
| -------------------------------- | ------------------------------ | ------------------------------------------------------------ |
| javax.net.ssl.keyStore           | FLUME_SSL_KEYSTORE_PATH        | Keystore location                                            |
| javax.net.ssl.keyStorePassword   | FLUME_SSL_KEYSTORE_PASSWORD    | Keystore password                                            |
| javax.net.ssl.keyStoreType       | FLUME_SSL_KEYSTORE_TYPE        | Keystore type(by default JKS)                                |
| javax.net.ssl.trustStore         | FLUME_SSL_TRUSTORE_PATH        | Truststore location                                          |
| javax.net.ssl.trustStorePassword | FLUME_SSL_TRUSTORE_PASSWORD    | Truststore password                                          |
| javax.net.ssl.trustStoreType     | FLUME_SSL_TRUSTORE_TYPE        | Trustore type(by default JKS)                                |
| flume.ssl.include.protocols      | FLUME_SSL_INCLUDE_PROTOCOLS    | Protocols to include when calculating enabled protocols. A comma separated list. Excluded protocols will be excluded from this list if provided. |
| flume.ssl.exclude.protocols      | FLUME_SSL_EXCLUDE_PROTOCOLS    | Protocols to exclude when calculating enabled protocols.A comma separated list. |
| flume.ssl.include.cipherSuites   | FLUME_SSL_INCLUDE_CIPHERSUITES | Cipher suites to include when calculating enabled cipher suites. A comman separated list. Excluded cipher suites will be excluded from this list if provided. |
| flume.ssl.exclude.cipherSuites   | FLUME_SSL_EXCLUDE_CIPHERSUITS  | Cipher suites to exclude when calculating enabled cipher suites.A comma separated list. |

SSL 属性既可以通过命令行设置，也可以在 conf/flume-env.sh 文件中通过 JAVA_OPTS 环境变量设置。(通过命令行设置是不太好的，因为命令中包含密码，会被保存在历史命令记录中)

```
export JAVA_OPTS="$JAVA_OPTS -Djavax.net.ssl.keyStore=/path/to/keystore.jks"
export JAVA_OPTS="$JAVA_OPTS -Djavax.net.ssl.keyStorePassword=password"
```

Flume 使用系统属性定义 JSSE (Java Secure Socket Extension), 这是一个标准的方法来设置 SSL。另一方面，在系统属性中设置密码意味着密码在进程列表中会被看到。在这个示例中，Flume 从对应的环境变量中初始化 JSSE 系统属性。

SSL 环境变量也可以设置 shell 脚本 conf/flume-env.sh 中。

```
export FLUME_SSL_KEYSTORE_PATH=/path/to/keystore.jks
export FLUME_SSL_KEYSTORE_PASSWORD=password
```

**注意：**

- SSL 在组件级别设置后，全局配置是无效的
- 如果在多个级别都设置了 SSL，优先级是这样的：
  1. agent 配置文件中的组件配置
  2. 系统属性
  3. 环境变量
- 如果开启 SSL 配置，但是没有通过任何方式设置 SSL 参数，那么会：
  - 在 keystore 中：configuration error
  - 在 truststores：the default trustore will be used(jssecacerts / cacerts in Oracle JDK)

## source 和 sink 的 batch size ，channel 的 transaction capacities

Source 和 sink 有一个 batch size 参数决定一批数据中最大 event 数。同时 channel 事务中有一个上限数叫 transaction capacity。Batch size 必须必 transaction capacities 小。在每次配置文件被读取的时候会检测配置是否正确。

# Flume Sources

## Avro Source

监听 Avro 端口，并从外部的 Avro 客户端接收 event。如果在上一级 agent使用 Avro Sink，就形成了多层级拓扑结构。

Avro Source 的属性(黑体字表示必须的属性)：

| **Property Name **        | **Default** | **Description**                                              |
| --------------------- | ----------- | ------------------------------------------------------------ |
| **channels**          |             |                                                              |
| **type**              | -           | 组件类型，avro                                               |
| **bind**              | -           | 监听机器的 hostname 或 IP                                    |
| **port**              | -           | 监听的端口                                                   |
| threads               | -           | 最大线程数                                                   |
| selector.type         |             |                                                              |
| selector.*            |             |                                                              |
| interceptors          | -           | 通过空格分隔的拦截器列表                                     |
| interceptors.*        |             |                                                              |
| compression-type      | none        | 属性值可以是 "none" 或 "deflate", 压缩类型必须跟 AvroSource 匹配 |
| ssl                   | false       | 这只为 true 将会开启 SSL 加密。如果开启 SSL，那么必须要指定 "keystore" 和 "keystore-password"，既可以通过组件级别的参数指定，也可以通过全局参数指定 |
| keystore              | -           | 这里指定 Java keystore 文件的路径。如果这里没有指定，会使用全局的 keystore(如果全局配置也没有指定，会 configuration error) |
| keystore-password     | -           | Java keystore 的密码。如果这里没有指定，会使用全局配置的 keystore password(如果全局配置也没有指定，会 configuratin error) |
| keystore-type         | JKS         | Java keystore 的类型，可以是 "JKS" 或 "PKCS12"。如果没有指定，会使用全局的配置(如果全局配置也没有指定，那使用默认配置 JKS) |
| exclude-protocols     | SSLv3       | 排除在外的 SSL/TLS 协议列表(通过空格分隔)，除了指定的协议外，SSLv3 会一直被排除在外 |
| include-protocols     | -           | 包含的 SSL/TLS 协议列表(通过空格分隔)，指定可用协议。如果为空，那么会包含所有可用的协议 |
| exclude-cipher-suites | -           | 排除的加密套件列表(通过空格分隔)                             |
| include-cipher-suites | -           | 包含的加密套件列表(通过空格分隔)                             |
| ipFilter              | false       | 设置为 true, 开启 ip 过滤                                    |
| ipFilterRules         | -           | 设置 ip 过滤的规则                                           |

示例：

```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141
```

ipFilterRules 示例：

ipFilterRules 使用逗号进行分隔，格式如下：

```
<'allow' or 'deny'>:<'ip' or 'name' for computer name>:<pattern>

示例：
ipFilterRules=allow:ip:127*,allow:name:name:localhost,deny:ip:*
```

> 配置 "allow:name:localhost,deny:ip:" 会允许本地的客户端，禁止其它 ip 的客户端
>
> 配置 "deny:name:localhost,allow:ip:" 会禁止本地的客户端而允许其它 ip 的客户端 

## Thrift Source

监听 Thrift 端口并接收来自 Thrift 客户端的数据，如果从另外一个 agent 的ThriftSink 获取数据，就创建了一个分层的拓扑结构。通过开启 kerberos 验证来开启 Thrift Source 的安全模式。agent-principal 和 agent-keytab 属性用来设置 Thrift Source 的 kerberos 认证。

Thrift Source 的属性(黑体字表示必须的属性)：

| PropertyName          | Default | Description                                                  |
| --------------------- | ------- | ------------------------------------------------------------ |
| **channels**          | -       |                                                              |
| **type**              | -       | 组件的类型, 需要设置为 *thrift*                              |
| **bind**              | -       | 监听的 ip 或 hostname                                        |
| **port**              | -       | 监听的端口                                                   |
| threads               | -       | 最大运行的线程数                                             |
| selector.type         |         |                                                              |
| selector.*            |         |                                                              |
| interceptors          | -       | 拦截器列表(通过空格分隔)                                     |
| interceptors.*        |         |                                                              |
| ssl                   | false   | 这只为 true 将会开启 SSL 加密。如果开启 SSL，那么必须要指定 "keystore" 和 "keystore-password"，既可以通过组件级别的参数指定，也可以通过全局参数指定 |
| keystore              | -       | 这里指定 Java keystore 文件的路径。如果这里没有指定，会使用全局的 keystore(如果全局配置也没有指定，会 configuration error) |
| keystore-password     | -       | Java keystore 的密码。如果这里没有指定，会使用全局配置的 keystore password(如果全局配置也没有指定，会 configuratin error) |
| keystore-type         | JKS     | Java keystore 的类型，可以是 "JKS" 或 "PKCS12"。如果没有指定，会使用全局的配置(如果全局配置也没有指定，那使用默认配置 JKS) |
| exclude-protocols     | SSLv3   | 排除在外的 SSL/TLS 协议列表(通过空格分隔)，除了指定的协议外，SSLv3 会一直被排除在外 |
| include-protocols     | -       | 包含的 SSL/TLS 协议列表(通过空格分隔)，指定可用协议。如果为空，那么会包含所有可用的协议 |
| exclude-cipher-suites | -       | 排除的加密套件列表(通过空格分隔)                             |
| include-cipher-suites | -       | 包含的加密套件列表(通过空格分隔)                             |
| kerberos              | false   | 设置为 *true* 开启 kerberos 认证。开启 kerberos 认证后，需要设置 *agent-principal* 和 *agent-keytab* 属性，并且只接受有 kerberos 认证的 thrift 客户端发送的数据 |
| agent-principal       | -       | principal 用来 Thrift Source 进行认证                        |
| agent-keytab          | -       |                                                              |

示例：

```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = thrift
a1.sources.r1.channel = c1
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141
```

## Exec Source

Exec Source 运行一个 Unix 命令，并获取命令的标准输出(错误输出会被忽略掉，除非 *logStdErr* 属性设置为 true)。如果程序一直存在，且 source 也一直存在，那么会产生更多的数据。也就是说像 `cat [named pipe]` 或 `tail -F [file]` 这样的命令会产生想要的结果，但是 `date `命令不会，前两个命令会产生一个数据流，而后一个命令只产生一条数据后就退出。

Exec Source 的属性(黑体字表示必须的属性)：

| PropertyName    | Default     | Description                                                  |
| --------------- | ----------- | ------------------------------------------------------------ |
| **channels**    | -           |                                                              |
| **type**        | -           | 组件的类型，这里需要设置为 `exec`                            |
| **command**     | -           | 要执行的命令                                                 |
| shell           | -           | 用来执行命令的 shell 类型，比如: /bing/sh -c。只有在执行的命令需要时才需要设置 |
| restartThrottle | 10000       | 在重试之前等待的时间(单位: ms)                               |
| restart         | false       | 如果执行命令挂掉后，是否重新执行                             |
| logStdErr       | false       | 是否输出命令的错误日志                                       |
| batchSize       | 20          | 一次读取并发送到 channel 的最大数据量                        |
| batchTimeout    | 3000        | 缓数据没有达到指定的大小，往下一个阶段发送钱等待的时间(单位：ms) |
| selector.type   | replicating | replicating / multiplexing                                   |
| selector.*      | -           | 根据 selector.type 决定                                      |
| interceptors    | -           | 拦截器列表(通过空格分隔)                                     |
| interceptors.*  |             |                                                              |

> ExecSource 和 其它异步 source 的问题是 source 不能保证如果数据往 channel 发送过程中失败的话，客户端是不知情的，这样数据就会丢失。最常见的是 `tail -F [file]` ，使用场景是一个应用的日志会输出到磁盘上的一个日志文件中，然后使用 flume tail 这个日志文件，每一行数据作为一个 event。有一个非常明显的问题就是，当 channel 满了后，不能再接收数据。flume 无法告诉应用程序再发送一次 channel 没有接收的数据。我们需要知道：在使用像 *ExecSource* 这样的单向的异步接口，应用程序无法保证数据被接收到。当使用这种 source 事，是完全无法保证数据被接收。如果需要保证高可靠性，可以考虑使用 *Spooling Directory Source*, *Taildir Source* 或直接通过 SDK 集成 Flume。

示例：

```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /var/log/secure
a1.sources.r1.channels = c1
```

*shell* 属性用来设置执行命令的 shell 类型(比如：Bash, Powershell)。*command* 属性值通过 shell 来执行。*shell* 属性常见的值有：’/bin/sh -c'，‘/bin/ksh -c'，’cmd /c'，'powershell -Command‘ 等。

```properties
a1.sources.tailsource-1.type = exec
a1.sources.tailsource-1.shell = /bin/bash -c
a1.sources.tailsource-1.command = for i in /path/*.txt; do cat $i; done
```

## JMS Source

JMS Source 从 类似于队列或主题这样的 JMS 目标中获取数据。作为一个 JMS 应用应该可以跟任何 JMS 生产者一块工作，但是这里只测试跟 ActiveMQ 一块工作。JMS Source 提供配置 batch size, message selector, user/pass以及 message 转换为 event。供应商提供的 JMS jar 包应该被包含在 Flume classpath 中，可以通过 plugin.d 目录，也可以在命令行添加 -classpath 参数，或者通过 flume-env.sh 文件中的 *FLUME_CLASSPATH* 变量来进行配置。

JMS Source 的属性(黑体字表示必须的属性)：

| Property Name             | Default | Descriptioni                                                 |
| ------------------------- | ------- | ------------------------------------------------------------ |
| **channels**              | -       |                                                              |
| **type**                  | -       | 组件的类型，这里应该是 `jms`                                 |
| **initialContextFactory** | -       | 初始化 Context Factory, 比如 org.apache.activemq.jndi.ActiveMQInitialContextFactory |
| **connectionFactory**     | -       | connection factory 中的JNDI(命名目录服务)                    |
| **providerURL**           | -       | JMS 提供者的 URL                                             |
| **destinationName**       | -       | 目标名称                                                     |
| **destinationType**       | -       | 目标类型(队列或主题)                                         |
| messageSelector           | -       | 创建消费者时使用的消息选择器                                 |
| userName                  | -       | 目标或提供者的用户名                                         |
| passwordFile              | -       | 包含目标或提供者密码的文件                                   |
| batchSize                 | 100     | 一批消费的消息数                                             |
| converter.type            | DEFAULT | 用来把消息转换成 event 的类                                  |
| converter.*               | -       | Converter 属性                                               |
| converter.charset         | UTF-8   | 转换 JMS 文本消息为字节数组是使用的字符集                    |
| createDurableSubscription | false   | 是否创建持久的订阅。持久的订阅只能用在目标类型为 topic。如果为 true, "clientId" 和 "durableSubscriptionName" 必须被指定 |
| clientId                  | -       | JMS 标识符，需要设置 durable subscriptions                   |
| durableSubscriptionName   | -       | 用来定义 durablesubscription 的名字，需要设置 durable subscriptions |

## Spooling Directory Source

这个 source 可以获取指定目录下文件中的数据，source 会监控指定的目录，当有新文件产生的时候，会把文件内容解析为 event。event 解析逻辑是插件化的。当一个文件被完全采集到 channel 后，默认情况下是重命名文件表明文件已经采集完成，也可以删除文件，或使用 trackerDir 来保持已经处理完文件的踪迹。

不像 Exec source，这个 source 是可靠的，不会丢失数据，甚至 Flume 重启或被 kill 也不会丢失数据。但是放入监控目录下的文件必须要快速，且文件名是唯一的。Flume 会察觉这些问题：

1. 文件放入监控目录后，还在继续写入内容，Flume 会打印错误到日志文件，并停止程序。
2. 如果文件名是重复的，Flume 会打印错误到日志文件，并停止程序。

为了避免这个问题，可以在文件移入监控目录是给文件名加一个唯一表示，比如时间戳。

尽管这个 source 是保证可靠性的，但是可能会出现 event 重复，如果下游发生错误。这个跟其它 flume 组件提供的保证是一致的。

| Property Name            | Default     | Description                                                  |
| ------------------------ | ----------- | ------------------------------------------------------------ |
| channels                 | -           |                                                              |
| type                     | -           | 组件的类型，这里需要是 spooldir                              |
| spoolDir                 | -           | 监控的目录                                                   |
| fileSuffix               | .COMPLETED  | 采集完文件的后缀                                             |
| deletePolicy             | never       | 什么时候删除文件：`never` 或 `immediate`                     |
| fileHeader               | false       | 是否添加一个header，来存储文件名的绝对路径                   |
| fileHeaderKey            | file        | 当需要添加文件名绝对路径到 event header 时，使用的 header key |
| basenameHeader           | false       | 是否添加一个 header，来存储文件名                            |
| basenameHeaderKey        | file        | 当需要添加文件名到 event header 时，使用的 header key        |
| includePattern           | ^.*$        | 用来指明哪些文件需要包含在内的正则表达式。它可以配合 `ignorePattern` 来使用，如果一个文件既匹配 `ignorePattern` 也匹配 `includePattern`，那这个文件会被忽略 |
| ignorePattern            | ^$          | 用来指明哪些文件被忽略                                       |
| trackerDir               | .flumespool | 指定一个目录来存储已经处理完文件的元数据。如果这个路径不是绝对路径，那它会被理解为 spoolDir 的相对路径 |
| trackingPolicy           | rename      | 指定已处理完的文件被追踪的策略，它可以是 `rename`，也可以是 `tracker_dir`。这个参数只有当 *deletePolicy* 是 *never* 时才起作用。`rename` - 文件被采集完后，根据 *fileSuffix* 参数来重命名文件。`tracker_dir` - 采集完的文件不会被重命名，但是一个空文件会在 trackerDir 中创建。这个新创建文件的名字通过 *fileSuffix* 来跟采集文件区别 |
| consumeOrder             | oldest      | 监控目录下文件被采集的顺序，`oldest`, `youngest`, `random`。就 `oldest` 和 `youngest` 这两个来说，会通过文件最后修改的时间来比较文件。如果时间相同，文件名词典顺序最小的会先被采集。如果使用 `random` 会随机采集文件。当使用 `oldest` 或 `youngest`，这个目录会被扫描来采集 oldest/youngest 文件，当有大量的文件时，采集会比较慢。但是使用 `random` 会造成新文件在不断产生是，较老的文件被最后采集 |
| pollDelay                | 500         | 轮询新文件的延迟(ms)                                         |
| recursiveDirectorySearch | false       | 是否监控子目录中新文件的产生                                 |
| maxBackoff               | 4000        | 如果 channel 满了的话，最大重试间隔。当 channel 抛出 ChannelException 时，source 每次会以指数方式增加时间，直到此参数指定的值 |
| batchSize                | 100         | 每批发往 channel 数据量                                      |
| inputCharset             | UTF-8       | 反序列化使用的字符集，将输入文件视为文本                     |
| decodeErrorPolicy        | FAIL        | 当我们在输入文件中看到一个无法解码的字符该怎么处理。`FAIL`：抛出一个异常，并且解析文件失败。`REPLACE`把无法解码的字符替换为 "replacement character" 字符，典型的编码格斯 U+FFFD。`IGNORE`：忽略无法解码的字符 |
| deserializer             | LINE        | 指定把文件转换为 event 的序列化工具。默认一行作为一个 event。指定的类必须实现 `EventDeserializer.Builder` |
| deserializer.*           |             | 每个事件反序列化器不同                                       |
| bufferMaxLines           | -           | 弃用                                                         |
| bufferMaxLineLength      | 5000        | 弃用，被 `deserializer.maxLineLength` 参数替代，用来指定一行数据最大长度 |
| selector.type            | replicating | replicating or multiplexing                                  |
| selector.*               |             | 依赖 selector.type 的值                                      |
| interceptors             | -           | 空格分隔的拦截器列表                                         |
| interceptor.*            |             |                                                              |

示例：

```properties
a1.sources = src-1
a1.sources.src-1.type = spooldir
a1.sources.src-1.channels = ch-1
a1.sources.src-1.spoolDir = /var/log/apache/flumeSpool
a1.sources.src-1.fileHeader = true
```

