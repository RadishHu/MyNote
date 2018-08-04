# Hadoop简介

## 目录

> * [Hadoop](#chapter1)
> * [HDFS](#chapter2)

## Hadoop <a id="chapter1"></a>

狭义的Hadoop是Apache的开源框架

- 核心组件有：
  - HDFS，分布式文件系统，解决海量数据存储
  - YARN，作业调度和集群资源管理的框架，解决资源任务调度
  - MapReduce，分布式运算编程框架，解决海量数据计算
- HDFS集群中的角色包括：NameNode、DataNode、SecondaryNameNode
- YARN集群中的角色包括：ResourceManager、NodeManager
- Hadoop部署方式：Standalone mode(独立模式)、Pseudo-Distributed mode(伪分布式模式)、Cluster mode(集群模式)

## HDFS <a id="chapter2"></a>

HDFS采用master/slave(主从)架构，NameNode是主节点，DataNode是从节点

HDFS中的文件在物理上时分块存储(block)，hadoop2.x默认块大小为128M，默认保存数量为3份

- 元数据

  目录结构及文件块位置信息叫元数据。NameNode负责维护整个hdfs文件系统的目录树结构，以及每一个文件对应的block块信息