# hive日志配置

- 修改Log4j配置文件

  Hive可以通过$HIVE_HOME/conf目录下的2个Log4j配置文件来配置日志：

  - hive-log4j.properties，控制CLI和其它本地执行组件的日志
  - hive-exec-log4j.properties，控制MapReduce task内的日志

- 临时修改日志配置

  Hive Shell启动时通过hiveconf参数指定log4j.properties文件中的任意属性。

  示例，指定输出日志为DEBUG级别，并输出到控制台：

  ```
  $bin/hive -hiveconf hive.root.logger=DEBUG,console
  ```

  