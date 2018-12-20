# Flume

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

