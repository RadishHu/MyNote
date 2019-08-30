# Apache Hadoop 2.7.7

## 概述

Apache Hadoop 2.7.7 是基于稳定版本 2.7 开发的，简要介绍下它的主要特征和提升：

- Common
  - 修复多个模块的 bug
  - 优化 GUI 分组控制
  - 提升压缩分片的阅读
- HDFS
  - 修复 NameNode 因为 NPE 和 full GC 崩盘的问题
  - 提升 NameNode 路径控制的性能
  - 提升命名系统
- YARN
  - Resource Manager 性能提升和 bug 修复
  - 控制 Node Manager 在 shuffle 过程中的超时问题

## 单节点部署

### 目的

这部分主要介绍如何搭建一个单节点的 Hadoop，以便可以快速上手 Hadoop MapReduce 和 Hadoop HDFS。

### 前期准备

- 平台
  - 可以使用 GNU/Linux 作为开发或生产环境，Hadoop 可以部署 2000 节点的 GNU/Linux 集群
  - 也可以在 windows 上部署
- 软件
  - jdk，jdk7 以上
  - ssh

## 集群部署

## Hadoop 命令

### 概述

所有的 hadoop 命令都在 bin/hadoop 目录下，执行 hadoop 脚本并不要带参数，会输出所有命令的描述。

命令格式：

```shell
hadoop [--config confdir] [--loglevel loglevel] [COMMAND] [GENERIC_OPTIONS] [COMMAND_OPTIONS]
```

> --config confdir 覆盖默认的配置文件，默认配置文件在 ${HADOOP_HOME}/conf 目录下
>
> --loglevel loglevel 设置日志级别，有效的日志级别有：FATAL, ERROR, WARN, INFO, DEBUG 和 TRACE, 默认的日志级别为 INFO

**Generic Options**

许多子命令都有一系列的配置项：

- -archives \<comma separated list of archives\>
- -conf \<configuration file\>，指定应用的配置文件
- -D \<property>=\<value>，设置指定的属性设置属性值
- -files \<comma sepearted list of file>，指定使用逗号分隔的文件列表，这些文件会被上传到 mapreduce 集群，仅适用于 job
- -jt \<local> or \<resourcemanager: prot>，指定一个 ResourceMamager
- -libjars \<comma seperated list of jars>，指定一个使用都好分隔的 jar 包列表，这些 jar 包会被包含在 classpath，仅适用于 job

### 用户命令

这些命令适用于 hadoop 集群的用户。

- archive

  创建一个 hadoop archive

- checknative

  命令格式：

  ```
  hadoop checknative [-a] [-h]
  ```

  > -a，检测所有可用的库
  >
  > -h，显示 help 信息

  这个命令检测 Hadoop 本地 code  的可用性。默认情况下，这个命令只检测 libhadoop 的可用性。

- classpath

  命令格式：

  ```
  hadoop classpath [--glob | --jar <path> | -h | --help]
  ```

  > --glob，扩展通配符
  >
  > --jar path，把 jar 中显示的 path 写为 classpath
  >
  > -h, --help，显示帮助信息

  打印 class path 需要获取到 hadoop jar 和必须的库。如果不使用任何参数，会输出命令脚本中设置的 classpath.

- credential

  命令格式：

  ```
  hadoop credential <subcommand> [options]
  ```

  > subcommand:
  >
  > - create alias [-provider provider-path]，提示用户证书以给定的别名进行保存。如果不指定 -provider，会使用 core-site.xml 文件的 hadoop.security.credential.provider.path 属性值。
  >
  > - delete alias [-provider provider-path] [-f]，删除给定别名的证书，如果不指定 -provider，会使用 core-site.xml 文件的 hadoop.security.credential.provider.path 属性值。如果不指定 -f，这个命令会要求验证。
  > - list [-provider provider-path]，证书别名的列表，如果不指定 -provider，会使用 core-site.xml 文件的 hadoop.security.credential.provider.path 属性值。

  这个命令是用来管理证书提供者的证书、密码。

  Hadoop 中的 CredentialProvider API 可以分隔 application 并定义如何存储必要的密码。为了指出特定的提供者的类型个和位置，用户必须指定 core-site.xml 文件中的 hadoop.security.credential.provider.path 属性或在命令中加入 -provider 选项。provider path 是一个通过逗号分隔的 URL 列表，这个 URL 列表指定了 provider 的类型和位置，比如：

  ```
  user:///,jceks://file/tmp/test.jceks,jceks://hdfs@nn1.example.com/my/path/test.jceks
  ```

  > 当前用的证书文件需要通过 User Provider 来查看，本地文件在 /tmp/test.jceks ，这是一个 java 密匙库提供者，并且 HDFS 的 nn1.example.com/my/path/test.jceks 也保存着一个 java 密匙库提供者。

  当使用 vredential 命令时，常常需要提供指定证书保存 provider 的密码。为了指明保存哪个 provider，需要使用 -provider 选项。当提供多个 provider 时，会使用第一个非暂时性的 provicder，示例：

  ```
  hadoop credential list -provider jceks://file/tmp/test.jceks
  ```

- distcp

  递归的复制文件或目录

- fs

  当在使用 HDFS 时，这个跟 hdfs/dfs 是同义的

- jar

  命令格式：

  ```
  hadoop jar <jar> [mainClass] args...
  ```

  运行一个 jar 包

- key

  命令格式：

  ```
  hadoop key <subcommand> [options]
  ```

  > subcommand:
  >
  > - create keyname [-cipher cipher] [-size size] [-description description] [-attrattribute=value] [-provider provider] [-help]，给 -*provider* 参数指定的 -*keyname*参数指定的创建一个新的 key。可以通过 -*cipher* 参数指定加密算法，默认的加密算法是 "AES/CTR/NoPadding"。默认的 keysize 是 128，可以通过 -*size* 参数指定 key 长度。attribute=value 格式的属性可以通过 -*attr*- 参数指定。
  > - roll keyname [-provider provider] [-help]，给 -*provider* 参数指定的 key 创建一个新的版本
  > - delete keyame [-provider provider] [-f] [-help]，删除 key 的所有版本，如果不适用 -*f* 这个命令需要验证
  > - list [-provider provider] [-metadata] [-help]， 
  > - -help
  
- trace

- version

  命令格式：

  ```
  hadoop version
  ```

  输出 hadoop 的版本

- CLASSNAME

  命令格式：

  ```shell
  hadoop CLASSNAME
  ```

  运行名为 CLASSNAME 的 class

### 管理员命令

hadoop 集群管理员适用的命令

- daemonlog

  命令格式：

  ```shell
  hadoop daemonlog -getlevel <host:httpport> <classname>
  hadoop daemonlog -setlevel <host:httpport> <classname> <level>
  ```

  > -getlevel host:httpport classname，输出指定运行在 *host:httpport* 后台程序的日志级别，这个命令实际上是连接到 *http://\<host:httpport>/logLevel?log=<classname\>*
  >
  > -setlevel host:httpport classname level，设置运行在 *http:httpport* 后台程序的日志级别，这个命令实际上是连接到 *http://\<host:httpport>/logLevel?log=\<classname>&level=\<level>

  获取/设置后台程序指定 classname 的日志级别

  示例：

  ```shell
  $ bin/hadoop daemonlog -setlevel 127.0.0.1:50070 org.apache.hadoop.hdfs.server.namenode.NameNode DEBUG
  ```


## FileSystem Shell

### 概述

文件系统(FS) shell 包括各种类似于 shell 命令，可以直接操作 Hadoop 分布式文件系统(HDFS) 和其它 Hadoop 支持的文件系统，比如：本地文件系统，HFTP 文件系统，S3 文件系统。FS shell 调用方式如下：

```shell
bin/hadoop fs <args>
```

所有的 FS shell 命令都以路径 URI 作为参数，URI 的格式为 *scheme://authority/path*，HDFS 的 scheme 是 `hdfs`，本地文件系统的 scheme 是 `file`。scheme 和 authority 都是可选的，如果没有指定 scheme，会使用配置文件中指定的。HDFS 中的文件或目录 /parent/child 可以指定为 hdfs://namenodehost/parent/child 或 /parent/child(配置文件中指定的是 hdfs://namenodehost)。

大多数 FS shell 命令跟 Unix 命令类似。如果使用的 HDFS，那么 hdfs 和 dfs 的意思是相近的。

### 常用命令

- appendToFile

  命令格式：

  ```sehll
  hadoop fs -appendToFile <localsrc> ... <dst>
  ```

  从本地文件系统追加单个 src 或多个 srcs 到目标文件系统。也可以从标准输入读取数据追加到目标文件系统。示例：

  - hadoop fs -appendToFile localfile /user/hadoop/hadoopfile
  - hadoop fs -appendToFile localfile1 localfile2 /user/hadoop/hadoopfile
  - hadoop fs -appendToFile localfile hdfs://nn.example.com/hadoop/hadoopfile
  - hadoop fs -appendToFile -hdfs://nn.example.com/hadoop/hadoop/hadoopfile Reads the input from stdin

  返回 0 表示成功，返回 1 表示失败。

- cat

  命令格式：

  ```shell
  hadoop fs -cat [ignorCrc] URI [URI ...]
  ```

  复制指定的文件到标准输出。

  选项：

  - -ignorCrc，不使用校验和验证

  示例：

  - hadoop fs -cat hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
  - hadoop fs -cat file:///file3 /user/hadoop/file4

  返回 0 表示成功，返回 1 表示失败。

- checksum

  命令格式：

  ```shell
  hadoop fs -checksum URI
  ```

  返回文件的校验和信息

  示例：

  - hadoop fs -checksum hdfs://nn1.example.com/file1
  - hadoop fs -checksum file:///etc/hosts
  
- chgrp

  命令格式：

  ```shell
  hadoop fs -chgrp [-R] GROUP URI [URI ...]
  ```

  改变文件所属的组，用户必须是文件的拥有者或者是超级用户。

  选项：

  - -R，递归修改目录下文件的所属组

- chmod

  命令格式：

  ```shell
  hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]
  ```

  修改文件的权限，用户必须是文件的所有者或者是超级用户。

  选项：

  - -R，递归修改目录下文件的权限

- chown

  命令格式：

  ```shell
  hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI]
  ```

  修改文件的拥有者，用户必须是超级用户。

  选项：

  - -R，递归修改目录下文件的权限

- copyFromLocal

  命令格式：

  ```shell
  hadoop fs -copyFromLocal <localsrc> URI
  ```

  类似于 fs -put 命令，除了文件源是受限制的，必须是本地文件。

  选项：

  - -p，保留访问和修改时间、所有权和权限(如果权限可以通过文件系统传播的话)
  - -f，如果文件已经存在的话，覆盖文件
  - -l，强行限制文件的副本数为1
  - -d，不创建临时文件，临时文件的后缀为 *.\_COPYING_*

- copyToLocal

  命令格式：

  ```shell
  hadoop fs -copyToLocal [-ignorecrc] [-crc] URI <localdst>
  ```

  类似于 get 命令，除了目标文件必须是本地文件

- count

  命令格式：

  ```shell
  hadoop fs -count [-q] [-h] [-v] <paths>
  ```

  统计目录和文件的个数和大小，该命令输出的列分别是：目录个数、文件个数、大小、路径名

  加上 -q 参数后输出的列分别是：配额，剩余的配额，空间配额，剩余的空间配额，目录个数，文件个数，大小，路径名

  -h 选项，以可读的格式显示大小

  -v 选项，显示每列的表头信息

  示例：

  - hadoop fs -count hdfs://nn1.example.com/file hdfs://nn2.example.com/file2
  - hadoop fs -count -q hdfs://nn1.example.com/file1
  - hadoop fs -count -q -h hdfs://nn1.example.com/file1
  - hdfs dfs -count -q -h -v hdfs://nn1.example.com/file1

  返回 0 表示成功，返回 1 表示失败。

- cp

  命令格式：

  ```shell
  hadoop fs -cp [-f] [-p | -p[topax]] URI [URI ...] <dest>
  ```

  从指定的源赋值文件到目的地。这个命令可以从多个数据源复制数据，这样做的话目的地必须是个目录。

  选项：

  - -f，如果目标文件存在，则覆盖目标文件
  - -p，保留文件属性(时间戳，所属权，权限，ACL，XAttr)，如果 -p 没有指定其他的参数，那么会保留文件的时间戳、所属权和权限。如果使用 -pa，也还保留权限，因为 ACL 是一组超级权限

  示例：

  - hadoop fs -cp /user/haoop/file1 /user/haoop/file2
  - hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir

  返回 0 表示成功，返回 1 表示失败。

- createSnapshot

- deleteSnapshot

- df

  命令格式：

  ```shell
  hadoop fs -df [-h] URI [URI ...]
  ```

  显示可用的空间

  选项：

  -h，以可阅读的格式显示大小

  示例：

  hadoop dfs -df /user/hadoop/dir1

- du

  命令格式：

  ```shell
  hadoop fs -du [-s] [-h] URI [URI ... ]
  ```

  如果指定的是目录，展示目录中文件和目录的大小；如果指定的是文件，展示文件的大小

  选项：

  - -s，展示聚合后的文件大小，而不是单个文件
  - -h，以可读的格式显示大小

- dus

  命令格式：

  ```shell
  hadoop fs -dus <args>
  ```

  显示聚合后的文件大小，**这个命令已经启动，可以使用 fs -du -s**

- expunge

  命令格式：

  ```shell
  hadoop fs -expunge
  ```

  永久删除检查点比 trash 目录设置的阈值更旧的文件，并创建一个新的检查点。当创建新的检查点，trash 目录中最近删除的文件且在检查点以下的会被移除。