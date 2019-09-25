# HBase Shell

# 进入HBase命令行：

```shell
$ ./bin/hbase shell [OPTIONS]
```

> OPTIONS:
>
> - -d, --debug, 设置日志级别为 DEBUG
> - -h, --help, 帮助页面
> - -n, --noninteractive, 

# 表管理

- 查看所有的表

  ```
  list
  ```

- 创建表

  ```
  create 'tableName',
  {NAME => 'familyName',VERSIONS => [versions]},
  {...},
  ...
  ```

  e.g.

  ```
  create 't1',
  {NAME => 'f1',VERSIONS => 2},
  {NAME => 'f2',VERSIONS => 2}
  ```

- 查看表结构

  ```
  describe 'tableName'
  ```

  e.g.

  ```
  describe 't1'
  ```

- 删除表

  ```
  #先disable表
  disable [tableName]
  
  #然后drop
  drop [tablename]
  ```

- 修改表的结构

  先停用表

  ```
  disabel 'tableName'
  ```

  添加列族

  ```
  alter 'tableName', NAME => 'familyName'
  ```

  删除列族

  ```
  alter 'tableName',NAME => 'familyName',METHON => 'delete'
  或
  alter 'tableName','delete' => 'familyName'
  ```

  修改列族版本号

  ```
  alter 'tableName',NAME => 'familyName',VRESIONS => 5
  ```

  最后启用表

  ```
  enable 'tableName'
  ```

# 表数据增删改查

- 添加数据

  ```
  put 'tableName','rowkey','familyName1:colunm1','value','timestamp'
  ```

  e.g. timestamp采用系统默认的

  ```
  put 't1','rowkey001','f1:col1','value01'
  ```

- 查询数据

  - 查询某行记录

    ```
    get 'tableName','rowkey','family:column',...
    ```

    e.g.

    获取rowkey为rk001的所有信息

    ```
    get 't1','rk001'
    ```

    获取rowkey为rk001，info列族的所有信息

    ```
    get 't1','rk001','info'
    ```

    获取rowkey为rk001，info列族的name、age列标识符的信息

    ```
    get 't1','rk001','info:name','info:age'
    ```

    获取rowkey为rk0001，info、data列族的信息 

    ```
    get 't1', 'rk0001', 'info', 'data'
    get 't1', 'rk0001', {COLUMN => ['info', 'data']}
    ```

    获取rowkey为rk0001，列族为info，版本号最新5个的信息 

    ```
    get 't1', 'rk0001', {COLUMN => 'info', VERSIONS => 5}
    get 't1', 'rk0001', {COLUMN => 'info:name', VERSIONS => 5}
    get 't1', 'rk0001', {COLUMN => 'info:name', VERSIONS => 5, TIMERANGE => [1392368783980, 1392380169184]}
    ```

    获取rowkey为rk0001，cell的值为zhangsan的信息 

    ```
    get 't1', 'rk0001', {FILTER => "ValueFilter(=, 'binary:zhangsan')"}
    ```

    获取rowkey为rk0001，列标示符中含有a的信息

    ```
    get 'people', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}
    ```

  - 扫描表

    ```
    scan 'tableName' ,{COLUMN => 'family:column'...,LIMIT => num}
    ```

    e.g.

    查询列族为info的记录，显示前5条数据

    ```
    scan 't1', {COLUMN => 'info'，LIMIT => 5}
    ```

    查询列族为info，列标识符为name的信息，并且版本最新的5个

    ```
    scan 't1',{COLUMN => 'info:name',VERSION => 5}
    ```

    查询列族为info和data，且列标识符中含有a字符的信息

    ```
    scan 't1',{COLUMN => ['info','data'],FILTER => "（QualifierFilter(=,'substring:a')）"}
    ```

    查询列族为info，rk范围是[rk001,rk003]的数据

    ```
    scan 't1',{COLUMN => 'info',STARTROW => 'rk001',ENDROW => 'rk003'}
    ```

    查询表中rowkey以rk字符开头的

    ```
    scan 't1',{FILTER => "PrefixFilter('rk')"}
    ```

    

- 删除数据

  - 删除行中某个列值

    ```
    delete 'tableName','rowkey','family:column','timestamp'
    #必须要指定列名
    ```

    e.g.删除表t1，rowkey001中的f1:col1的数据

    ```
    delete 't1','rowkey001','f1:col1'
    ```

  - 删除行

    ```
    deleteall 'table','rowkey'，'family:column','timestamp'
    #可以不指定列名，删除整行数据
    ```

  - 删除表中所有数据

    ```
    truncate 'tableName'
    ```

# 退出 Shell

通过 `quit` 退出