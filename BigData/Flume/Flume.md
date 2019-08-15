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

`event` 是 Flume 数据流的基本单元，它包含一个字节的有效负载和一个可选的属性集合

`agent` 是一个 JVM 进程，它包含 `source`, `channel`, `sink` 等组件，它利用这些组件将 `event` 从数据源传送到目的地(或另一个节点)













Flume核心概念：

- Event
- Client
- Agent
  - Sources、Channels、Sinks
  - 其它组件：Interceptors、Channel Selectors、Sink Processor

## Event

Event是flume数据传输的基本单元，flume以事件的形式将数据从源头传动到最终目的

Event由可选的headers和载有数据的一个byte array构成

## Client

Client将原始log包装成events并且发送它们到一个或多个agent的实体

## Agent

一个Agent包含Sources、Channels、Sinks和其它组件，它利用这些组件将events从一个节点传输到另一个节点或最终目的

agent是flume流的基础部分

