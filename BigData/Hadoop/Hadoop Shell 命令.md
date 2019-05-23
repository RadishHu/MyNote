# Hadoop Shell 命令

命令格式：

```
hadoop fs <args>
```

fs shell 命令使用 URI 路径作为参数，URI 格式为 scheme://authority/path

> - 对于 HDFS 文件系统，scheme 是 `hdfs`；对于本地文件系统，scheme 是 `file`
>
> - scheme 和 authority 是可选的，如果没有指定就会使用配置中指定的默认 shceme
>
>   比如：一个 HDFS 文件 /parent/child 可以表示成 hdfs://namenode:namenodeport/parent/child，如果配置文件中的默认值是 namenode:namenodeport，可以简写为 /parent/child



FS Shell 可以使用的命令：

- cat

  查看指定文件的内容

  ```
  hadoop fs -cat URI [URI...]
  ```

- chgrp

  改变文件所属的组

  ```
  hadoop fs -chgrp [-R] GROUP URI [URI...]
  ```

  > - -R 在指定的目录结构内进行递归修改所属组

- chmod

  改变文件的权限

  ```
  hadoop fs -chmod [-R] <MODE [,MODE]... | OCTALMODE> URI [URI...]
  ```

  > - -R 在指定的目录结构内进行递归修改文件权限

- chown

  改变文件的拥有者

  ```
  hadoop fs -chown [-R] [OWNER] [:[GROUP]] URI [URI]
  ```

  > - -R 在指定的目录结构内进行递归修改文件拥有者





参考：

- [《FS Shell 使用指南》](<https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html>)