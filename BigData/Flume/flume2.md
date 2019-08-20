## 在配置文件中使用环境变量

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

## 记录原始数据

在生产环境日志中输出原始数据流不是一个明智之举，这样会在 flume 日志中泄露一些敏感的信息。flume 不会在日志中输出这样的信息。

如果想要在日志中输出 event 或配置相关的数据，可以配置 log4j 的属性和一些 java 属性：

- 输出配置相关的数据，可以设置 java 的 `-Dorg.apache.flume.log.printconfig=true` 属性，这个可以在启动命令里设置，也设置 flume-env.sh 文件中的 `JAVA_OPTS` 变量。
- 输出 event 数据，可以通过启动命令设置 java 的 `-Dorg.apache.flume.log.rawdata=true` 属性，同时还必须设置 log4j 的日志输出级别为 `DEBUG` 或 `TRACE` 来使 event 数据输出到 flume 日志中。

示例：

```shell
$ bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=DEBUG,console -Dorg.apache.flume.log.printconfig=true -Dorg.apache.flume.log.rawdata=true
```

> 通过这个启动命令，可以在 flume 日志中看到 event 和配置相关的信息

## 使用 Zookeeper 保存配置文件

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

## 安装第三方插件

