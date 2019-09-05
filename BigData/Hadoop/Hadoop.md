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

  显示聚合后的文件大小，**这个命令已经弃用，可以使用 fs -du -s**

- expunge

  命令格式：

  ```shell
  hadoop fs -expunge
  ```

  永久删除 trash 目录中检查点比设置的阈值更旧的文件，并创建一个新的检查点。当创建新的检查点，trash 目录中最近被删除的文件会被移动到检查点下。检查点比 *fs.trash.checkpoint.interval* 更旧的文件会下一次执行 *-expunge* 时被永久的删除。
  
  用户可以通过 *core-site.xml* 文件中的 *fs.trash.checkpoin.interval* 属性来设置创建和删除检查点的周期，这个配置的值应该小于等于 *fs.trash.interval* 属性的值。
  
- find

  命令格式：

  ```shell
  hadoop fs -find <path> ... <expression> ...
  ```

  查找所有跟指定的表达式匹配的文件。如果没有指定 *path* 则默认使用当前目录。如果没有指定 *expression* 表达式，则默认使用 *-print*。

  可以使用的 *expression* :

  - -name pattern，如果文件的名字跟指定的模型一样则标记为 true

    -iname pattern，忽略大小写

  - -print，标记为 true，且当前的目录路径会被写在标准输出中。

    -print0Always，附加一个 ASCII NULL 字符

  也可以使用以下的表达式：

  - -a / -and，逻辑运算符 *AND*，用来关联两个表达式。如果两个子表达式都返回 true 则返回 true。

  示例：

  ```shell
  hadoop fs -find / -name test -print
  ```

- get

  命令格式 ：

  ```shell
  hadoop  fs -get [ignorecrc] [-crc] [-p] [-f] <src> <localdst>
  ```

  复制文件到本地文件系统。使用 -*crc* 选项，文件和 CRCs 都会被复制。使用 *-ignorecrc* 选项，CRC 验证失败的额文件会被复制。

  示例：

  - hadoop fs -get /user/hadoop/file localfile
  - hadoop fs -get hdfs://nn.example.com/user/hadoop/file localfile

  选项：

  - -p，保留创建和修改时间，所属关系和权限。(如果权限可以在文件系统间传递的话)
  - -f，如果目标文件存在的话，覆盖目标文件
  - -ignorecrd，下载文件是跳过 CRC 检测
  - -crc，下载文件时写下 CRC 检测和

- getfacle

  命令格式：

  ```shell
  hadoop fs -getfacl [-R] <path>
  ```

  显示文件和目录的权限列表(ACLs)，如果一个目录有默认的 ACL，那么也会显示出默认的 ACL。

  选项：

  - -R，递归的显示所有文件和目录的 ACL 列表
  - path，要显示的文件或目录

  示例：

  - hadoop fs -getfacl /file
  - hadoop fs -getfacl -R /dir

- getfattr

  命令格式：

  ```shell
  hadoop  fs -getfattr [-R] -n name | -d [-e en] <path>
  ```

  展示文件或目录的所有属性

  选项：

  - -R，递归的展示所有文件和目录的属性
  - -n name，输出名为 name 的扩展属性
  - -d，输出所有跟路径名关联的扩展属性
  - -e encoding，获取到属性值都对其进行编码。有效的编码格式有: text, hex 和 base64。
  - path，指定文件或目录

- getmerge

  命令格式：

  ```shell
  hadoop fs -getmerge [-nl] <src> <localdst>
  ```

  合并指定目录中的文件到指定的本地目录中，指定 *-nl* 选项，会在每个文件内容后添加一个换行符(LF)。

  示例：

  - hadoop fs -getmerge -nl /src /opt/output.txt
  - hadoop fs -getmerge -nl /src/file.txt /src/file2.txt /output.txt

- help

  命令格式：

  ```shell
  hadoop fs -help
  ```

  返回使用说明

- ls

  命令格式：

  ```shell
  hadoop fs -ls [-d] [-h] [-R] <args>
  ```

  选项：

  - -d，以普通文件的格式列出目录
  - -h，以可读的格式展示文件大小
  - -R，递归的列出子目录中的文件

  对文件使用 *ls* 命令会返回文件的信息，格式如下：
  
  ```
  permissions number_of_replicas userid groupid filesize modification_date modification_time filename
  ```
  
  对目录使用 *ls* 命令会返回它的直接子目录，格式如下：
  
  ```
  permissions userid groupid modification_date modification_time dirname
  ```
  
  > 默认情况下，目录中的文件是以文件名进行排序
  
  示例：
  
  - hadoop fs -ls /user/hadoop/file1
  
- lsr

  命令格式：

  ```shell
  hadoop fs -lsr <args>
  ```

  *ls* 命令的递归版本，这个命令已经弃用，使用 *hadoop fs -ls -R* 替代

- mkdir

  命令格式：

  ```shell
  hadoop fs -mkdir [-p] <paths>
  ```

  创建目录

  选项：

  - -p，这个选项的作用跟 unix 中的 mkdir -p 命令类似，会创建父级的目录

  示例：

  - hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2
  - hadoop fs -mkdir hdfs://nn1.example.com/user/hadoop/dir hdfs://nn2.example.com/user/hadoop/dir

- moveFromLocal

  命令格式：

  ```shell
  hadoop fs -moveFromLocal <localsrc> <dst>
  ```

  类似于 put 命令，从本地复制文件后，删除源文件

- moveToLocal

  命令格式：

  ```shell
  hadoop fs -moveToLocal [-crc] <src> <dst>
  ```

  显示 "Not implemented yet" 信息

- mv

  命令格式：

  ```shell
  hadoop fs -mv URI [URI ... ] <dest>
  ```

  移动文件，这个命令可以同时移动多个文件，但是要求目的地是一个目录。跨文件系统移动文件是不被允许的。

  示例：

  - hadoop fs -mv /user/hadoop/file1 /user/hadoop/file2
  - hadoop fs -mv hdfs://nn.example.com/file1 hdfs://nn.example.com/file2 hdfs://nn.example.com/file3 hdfs://nn.example.com/dri1

- put

  命令格式：
  
  ```shell
  hadoop fs -put [-f] [-p] [-l] [-d] [- | <localsrc1> ...] <dst>
  ```
  
  复制单个或多个本地文件到目标文件系统中。如果 source 设置为 "-" 的话，是从标准输入中读取数据写入目标文件系统。
  
  选项：
  
  - -p，保留访问和修改时间、所属关系和权限
  - -f，如果文件已经存在，覆盖文件
  - -l，允许 DataNode 懒惰的保存文件到磁盘，强行将副本数设置为 1
  - -d，跳过创建临时文件 .\_COPYING_ 的步骤
  
  示例：
  
  - hadoop fs -put localfile /user/hadoop/hadoopfile
  - hadoop fs -put -f localfile localfile2 /user/hadoop/hadoopdir
  - hadoop fs -put -d localfile hdfs://nn.example.com/hadoop/hadoopfile
  - hadoop fs -put - hdfs://nn.example.com/hadoop/hadoopfile Reads the input from stdin.
  
- renameSnapshot

- rm

  命令格式：

  ```shell
  hadoop fs -rm [-f] [-r | -R] [-skipTrash] URI [URI ...]
  ```

  删除指定的 文件，开启 trash 后，文件系统把要删除的文件移动到 trash 目录，而不是直接 删除。trash 默认是关闭的，用户可以通过设置 *fs.trash.interval* (core-site.xml) 属性为大于 0 的值来开启这个功能。

  选项：

  - -f，当文件不存在时，不显示报错信息或退出状态
  - -R，递归的 删除目录中的内容
  - -r，等价于 -R
  - -skipTrash，忽略 trash，直接删除文件。用于从一个超过配额的的目录中删除文件。

  示例：

  - hadoop fs -rm hdfs://nn.example.com/file /user/hadoop/emptydir

- rmdir

  使用示例：

  ```shell
  hadoop fs -rmdir [--ignore-fail-on-non-empty] URI [URI ...]
  ```

  删除目录

  选项：

  - --ignore-fail-on-non-empty，当使用通配符时，如果目录中还有文件，不会失败

  示例：

  - hadoop fs -rmdir /user/hadoop/emptydir

- rmr

  命令格式：

  ```shell
  hadoop fs -rmr [-skipTrash] URI [URI ...]
  ```

  递归删除文件，这个命令已经被弃用，使用 *hadoop fs -rm -r* 替代

- setfacl

  命令格式：

  ```shell
  hadoop fs -setfacl [-R] [-b | -k -m | -x <acl_spec> <path>] | [--set <acl_spec> <path>]
  ```

  设置文件和目录的权限控制列表(ACLs)

  选项：

  - -b，移除基础 ACL 外的所有权限控制。用户、分组和其它兼容性权限会保留
  - -k，移除默认的 ACL
  - -R，递归的应用操作到所有的文件和目录
  - -m，修改 ACL，新的条目会增加到 ACL，已经存在的条目继续保存
  - -x，移除指定的 ACL 条目，其它的 ACL 条目保存
  - --set，完全替换 ACL，完全抛弃已存在的条目。*acl_apec* 必须包含用户、分组和其它兼容性的条目
  - acl_apec，用逗号分隔的 ACL 条目
  - path，要修改的文件或目录

  示例：

  - hadoop fs -setfacl -m user:hadoop:rw- /file
  - hadoop fs -setfacl -x user:hadoop /file
  - hadoop fs -setfacl -b /file
  - hadoop fs -setfacl -k /dir
  - hadoop fs -setfacl --set user::rw-,user:hadoop:rw-,group::r--,other::r-- /file
  - hadoop fs -setfacl -R -m user:hadoop:r-x /dir
  - hadoop fs -setfacl -m default:user:hadoop:r-x /dir

- setfattr

  命令格式：

  ```shell
  hadoop fs -setfattr -n name [-v value] -x name <path>
  ```

  给文件或目录设置一个扩展的属性名称和值

  选项：

  - -b，移除基础 ACL 条目以外的所有条目。保留用户、分组和其它条目来跟权限兼容
  - -n name，扩展属性的名称
  - -v value，扩展属性的值，对这个值有三种不同的编码方式
  - -x name，移除扩展属性
  - path，文件或目录

  示例：

  - hadoop fs -setfattr -n user.myAttr -v myValue /file
  - hadoop fs -setfattr -n user.noValue /file
  - hadoop fs -setattr -x user.myAttr /file

- setrep

  命令格式：

  ```shell
  hadoop fs -setrep [-R] [-w] <numReplicas> <path>
  ```

  修改文件的副本数，如果 *path* 是一个目录，这个命令会递归的修改这个目录下所有文件的副本数。

  选项：

  - -w，命令等待复制完成，这个可能会小号很长的时间
  - -R，接受回退兼容性

  示例：

  - hadoop fs -setrep -w 3 /user/hadoop/dir1

- stat

  命令格式：

  ```shell
  hadoop fs -stat [format] <path>
  ```

  以指定的格式输出文件或目录的统计资料，*format* 可以是：块的文件大小(%b)，类型(%F)，所属组名(%g)，名字(%n),块大小(%o)，副本数(%r)，所属用户名(%u)和修改日期(%y,%Y)。%y 显示的 "yyyy-MM-dd HH:mm:ss" 格式，%Y 显示的是从 1970年1月1号到当前的毫秒数。如果没有指定 *format* ，默认使用 %y

  示例：

  - hadoop fs -stat "%F %u:%g %b %y %n" /file

- tail

  命令格式：

  ```shell
  hadoop fs -tail [-f] URI
  ```

  显示文件最新的输出

  选项：

  - -f，随着文件大小增长，不但输出文件新增的内容

  示例：

  - hadoop fs -tail pathname

- test

  命令格式：

  hadoop fs -test -[defsz] URI

  选项：

  - -d，如果指定的路径是个目录，返回 0
  - -e，如果指定的路径存在，返回 0
  - -f，如果指定路径是一个文件，返回 0
  - -s，如果指定的路径是空的，返回 0
  - -z，如果文件大小为 0，返回 0

  示例：

  - hadoop fs -test -e filename

- text

  命令格式：

  ```shell
  hadoop fs -text <src>
  ```

  以文本的格式输出文件的内容

- touchz

  命令格式：

  ```shell
  hadoop fs -touchz URI [URI ...]
  ```

  创建一个0长度的文件，如果文件已经存在 会报错

  示例：

  - hadoop fs -touchz pathname

- truncate

  命令格式：

  ```shell
  hadoop fs -truncate [-w] <length> <paths>
  ```

  截取匹配的文件到指定的大小

  选项：

  - -w，要求命令等待块恢复完成，如果不使用 -w，文件可能会 保持未关闭的一段时间，在这段时间内文件不能再次打开

  示例：

  - hadoop fs -truncate 55 /user/hadoop/file1 /user/hadoop/file2
  - hadoop fs -truncate -w 127 hdfs://nn1.example.com/user/hadoop/file1

- usage

  hadoop fs -usage command

  返回指定命令的帮助信息

### 对象存储

Hadoop FileSystem shell 工作在对象存储上，比如 S3、WASB 和 Swift。

```shell
# 创建一个目录
hadoop fs -mkdir s3a://bucket/datasets/

# 上传文件
hadoop fs -put /datasets/example.orc s3a://bucket/datasets/

# 创建一个文件
hadoop fs -touchz wasb://yourcontainer@youraccount.blob.core.windows.net/touched
```

不像通常文件系统，重命名对象存储中的文件或目录所消耗的时间跟操作对象的大小成正比 。因此很多文件系统的 shell 操作把重命名作为操作的最后阶段，跳过这个阶段可以降低延迟。

使用 *put* 和 *copyFromLocal* 命令应该添加 *-d* 选项。

```shell
# 从集群文件系统上传文件
hadoop fs -put -d /datasets/example.orc s3a://bucket/datasets/

# 从本地文件系统上传文件
hadoop fs -copyFromLocal -d -f /datasets/devices.orc s3a://bucket/datasets

# 从标准输入创建文件
# "-" 意思是用户标准输入
echo "hello" | hadoop fs -put -d -f - wasb://yourcontainer@youraccount.blob.core.windows.net/hello.txt
```

对象可以被下载和查看：

```shell
# 复制一个目录到本地文件系统
hadoop fs -copyToLocal s3a://bucket/datasets

# 从对象存储复制一个文件到集群文件系统
hadoop fs -get wasb://yourcontainer@youraccount.blob.core.windows.net/hello.txt /examples

# 打印对象
hadoop fs -cat wasb://yourcontainer@youraccount.blob.core.windows.net/hello.txt

# 打印对象，如果必要的话并解压它
hadoop  fs -text wasb://yourcontainer@youraccount.blob.core.windows.net/hello.txt

# 下载日志文件到本地文件
hadoop fs -getmerge wasb://yourcontainer@youraccount.blob.core.windows.net/logs\* log.txt
```

列出许多文件的命令会明显比运行在 HDFS 或其他文件系统上慢：

```shell
hadoop fs -count s3a://bucket/
hadoop fs -du s3a://bucket
```

> 其它比较慢的命令有： find, mv, cp 和 rm

## Hadoop 兼容性

# 通用

## 机架感知

为了提高 HDFS 的容错性，需要根据机架感知将一个块的副本放在不同的机架上。

Hadoop master 进程通过调用配置文件指定的外部脚本或 java 类来获取 slaves 的机架 id。要获取网络拓扑，java 类或外部脚本必须实现 java 的 **org.apache.hadoop.net.DNSToSwitchMapping** 接口。这个接口会维护一个一对一的通行，拓扑信息的格式为 */myrack/myhost*，这里的 '/' 是分隔符，'myrack' 是机架表示符，'myhost' 是单个的 host。假设每个机架有24个子网，可以使用 '/192.168.100.0/192.168.100.5' 表示一个机架-主机拓扑映射关系。

使用 java class 获取拓扑关系，需要在配置文件中指定 **topology.node.switch.mapping.impl** 参数。比如，*NetworkTopology.java* 被包含在 hadoop 集群中。使用 java 类跟使用外部的脚本相比有一个 性能优势，就是一个新的 slave 注册到集群中时不需要单独分出一个进程来。

如果使用一个外部的脚本，需要在配置文件中指定 **topology.script.file.name** 参数。不像 java 类，外部脚本不被包含在 hadoop 集群中，需要 hadoop 管理员提供。在使用 外部脚本时，Hadoop 需要向 ARGV 发送多个 IP 地址，IP 地址的数量由 **net.topology.script.numbers.args**  参数控制，默认为 100个。如果 **net.topology.script.numbers.args** 被修改为 1，拓扑脚本会向 DataNode 或 NodeManager 提交每个 IP。

如果 **topology.script.file.name** 或 **topology.node.switch.mappig.impl** 没有被设置，那返回的每个 IP 地址的机架 id 都会为 '/default-rack'。虽然这种行为是可取的，但是可能会导致 HDFS 副本出现问题，默认情况是将块的副本写入到 分散的机架中，但是这是无法实现的，因为只有一个机架名为 '/default-rack'。

另外一个配置参数是 **mapreduce.jobtracker.taskcache.levels**，这个参数决定 MapReduce 缓存在网络拓扑中的级别。比如，如果使用默认值 2，会构建一个两级的缓存，一个用于主机(主机-任务映射)，另一个用于机架(机架-任务映射)。

### python 示例

```python
#!/usr/bin/python
# this script makes assumptions about the physical environment.
#  1) each rack is its own layer 3 network with a /24 subnet, which
# could be typical where each rack has its own
#     switch with uplinks to a central core router.
#
#             +-----------+
#             |core router|
#             +-----------+
#            /             \
#   +-----------+        +-----------+
#   |rack switch|        |rack switch|
#   +-----------+        +-----------+
#   | data node |        | data node |
#   +-----------+        +-----------+
#   | data node |        | data node |
#   +-----------+        +-----------+
#
# 2) topology script gets list of IP's as input, calculates network address, and prints '/network_address/ip'.

import netaddr
import sys
sys.argv.pop(0) # discard name of topology script from argv list as we just want IP addresses

netmask = '255.255.255.0' # set netmask to what's being used in your environment.  The example uses a /24

for ip in sys.argv: # loop over list of datanode IP's
	address = '{0}/{1}'.format(ip, netmask) # format address string so it looks like 'ip/netmask' to make netaddr work
try:
   network_address = netaddr.IPNetwork(address).network # calculate and print network address
   print "/{0}".format(network_address)
except:
   print "/rack-unknown" # print catch-all value if unable to calculate network address
```

> 这个脚本接受一个参数，输出一个值，接受的参数通常为 DataNode 机器的 ip 地址，输出的值通常为该 DataNode 机器所在的机架 id。

### bash 示例

```shell
#!/bin/bash
# Here's a bash example to show just how simple these scripts can be
# Assuming we have flat network with everything on a single switch, we can fake a rack topology.
# This could occur in a lab environment where we have limited nodes,like 2-8 physical machines on a unmanaged switch.
# This may also apply to multiple virtual machines running on the same physical hardware.
# The number of machines isn't important, but that we are trying to fake a network topology when there isn't one.
#
#       +----------+    +--------+
#       |jobtracker|    |datanode|
#       +----------+    +--------+
#              \        /
#  +--------+  +--------+  +--------+
#  |datanode|--| switch |--|datanode|
#  +--------+  +--------+  +--------+
#              /        \
#       +--------+    +--------+
#       |datanode|    |namenode|
#       +--------+    +--------+
#
# With this network topology, we are treating each host as a rack.  This is being done by taking the last octet
# in the datanode's IP and prepending it with the word '/rack-'.  The advantage for doing this is so HDFS
# can create its 'off-rack' block copy.
# 1) 'echo $@' will echo all ARGV values to xargs.
# 2) 'xargs' will enforce that we print a single argv value per line
# 3) 'awk' will split fields on dots and append the last field to the string '/rack-'. If awk
#    fails to split on four dots, it will still print '/rack-' last field value

echo $@ | xargs -n 1 | awk -F '.' '{print "/rack-"$NF}'
```

### 查看集群机架信息

```shell
hdfs dfsadmin -printTopology
```

# HDFS

## HDFS 用户指南

这部分文档开始介绍 Hadoop Distributed File System(HDFS) 的使用。

### 概述

HDFS 是 一个分布式存储应用。HDFS 主要由 NameNode 和 DataNode 组成，NameNode 用来存储文件的元数据，DataNode 用来存储文件。