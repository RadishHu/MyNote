# Hive 变量



Hive 有几种变量：hiveconf、system、env、hivevar

- hiveconf 指 hive-site.xml 文件中配置的变量
- system 指系统变量，包括 JVM 的运行环境
- env 值环境变量，包括 Shell 环境下的变量信息，如 HIVE_HOME 之类的
- hivevar 自定义变量

## 变量声明

- 通过 `--define key=value` 或者 `--hivevar key=value` 来声明变量：

  ```shell
  $ hive --define a='I'
  # 简写
  $ hive -d key=value
  # 或者
  $ hive --hivevar key=value
  ```

- 声明多个变量

  ```shell
  $ hive --define a='I' --define b='love'
  ```

- 在 Hive 客户端，通过 `set` 来定义变量

  ```
  hive > set c='you';
  ```

## 引用变量

- 查看变量，通过 `set`

  只输入 set 会输出所有的变量

  要查看某一个变量的值可以通过 set varName:

  ```
  hive > set b;
  b='love'
  ```

- 通过 `${hiveconf:val}` 来引用变量：

  ```
  hive > select * from employees where name='${hiveconf:val}'
  ```

  