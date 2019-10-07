# 日志聚合

通过 **yarn.log-aggregation-enable** 开启日志聚合后，container 的日志会从本地磁盘复制到 HDFS ，并从本地删除。聚合后的日志在 HDFS 的存储路径由 **yarn.nodemanager.remote-app-log-dir** 和 **yarn.nodemanager.remote-app-log-dir-suffix** 进行设置，日志存储路径为 {yarn.nodemanager.remote-app-log-dir}/${user}/{yarn.nodemanager.remote-app-log-dir-suffix}。

查看聚合后日志的方式：

- 通过 *yarn logs* 命令在集群的任何节点查看聚合后的日志：

  ```shell
  yarn logs --applicationId <app ID>
  ```

- 可以直接使用 HDFS shell 或 API 查看 container 日志文件，日志在 HDFS 的存储路径由 **yarn.nodemanager.remote-app-log-dir** 和 **yarn.nodemanager.remote-app-log-dir-suffix** 进行设置，日志存储路径为 {yarn.nodemanager.remote-app-log-dir}/${user}/{yarn.nodemanager.remote-app-log-dir-suffix}。
- 如果是要查看 Spark on YARN 的 spark 日志，可以通过 Spark Web UI 下的 Executor 菜单页查看，需要开启 Spark history 和 MapReduce history 服务。

如果没有开启聚合日志，日志在本地磁盘的保存目录通过 **yarn.nodemanager.log-dirs** 来进行设置，日志会保存在 container 所在的机器上，目录结构为 {yarn.nodemanager.log-dirs}/{Application ID}/{container ID}