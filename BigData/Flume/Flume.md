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

