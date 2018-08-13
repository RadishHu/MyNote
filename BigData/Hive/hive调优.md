# hive调优

## 目录

> * [hive join调优](#chapter1)
> * [并行执行](#chapter2)
> * [严格模式](#chapter3)
> * [JVM重用](#chapter4)

## hive join调优 <a id="chapter1"></a>

[hive join调优](/BigData/Hive/hive_join调优.md)

## 并行执行 <a id="chapter2"></a>

Hive会将一个查询转换为一个或多个阶段，这样的阶段可以是MR阶段、抽样阶段、合并阶段、limit阶段等。Hive默认情况下一次只会执行一个阶段，当某个job包含多个阶段，并且这些阶段不完全相互依赖，那么这些阶段可以并行执行

开启并发执行：

```xml
<property>
    <name>hive.exec.parallel</name>
    <value>true</value>
    <description>Whether to execute jobs in parallel</description>
</property>
```

## 严格模式 <a id="chapter3"></a>

设置严格模式：

```xml
<property>
    <name>hive.mapred.mode</name>
    <value>strict</value>
</property>
```

通过严格模式可以禁止3种类型的查询：

- 对于分区表，必须在Where语句中包含分区字段过滤条件来限制数据范围，否则不允许执行
- 对于ORDER BY语句的查询，必须使用LIMIT语句，因为ORDER BY会将所有的结果数据分发到同一个reducer中进行处理
- 限制进行笛卡尔积的查询

## JVM重用 <a id="chapter4"></a>

JVM重用是Hadoop调优的内容，Hadoop默认通过使用派生JVM来执行map和reduce任务，这时JVM启动过程会造成比较大的开销，JVM重用可以使JVM实例在同一个job中重用N次

JVM重用在Hadoop的mapred-site.xml文件中进行设置：

```xml
<property>
    <name>mapred.job.reuse.jvm.num.tasks</name>
    <value>10</value>
    <description>How many tasks to run per jvm.If set to -1,there is no limit.</description>
</property>
```

