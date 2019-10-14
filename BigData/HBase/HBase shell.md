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

- 列出所有的表

  ```
  list
  ```

- 创建表

  ```
  create 'tableName', 'columnFamilyName1', 'columnFamilyName2' ...
  或
  create 'tableName',
  {NAME => 'columntFamilyName1',VERSIONS => [versions]},
  {...},
  ...
  ```

  e.g.

  ```
  create 't1', 'f1', 'f2'
  或
  create 't1',
  {NAME => 'f1',VERSIONS => 2},
  {NAME => 'f2',VERSIONS => 2}
  ```

- 查看表结构

  ```
  desc 'tableName'
  或
  describe 'tableName'
  ```

- 删除表

  ```
  #先 disable 表
  disable 'tableName'
  
  #然后 drop
  drop 'tablename'
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
  alter 'tableName','delete' => 'familyName'
  或
  alter 'tableName',NAME => 'familyName',METHON => 'delete'
  ```

  修改列族版本数

  ```
  alter 'tableName', {NAME => 'familyName',VERSIONS => 5}
  ```

  最后启用表

  ```
  enable 'tableName'
  ```

- 清空表数据

  ```
  truncate 'tableName'
  ```

- 查看表是否存在

  ```
  exists 'tableName'
  ```

- 查看表是否被禁用/启用

  ```
  is_disable 'tableName'
  is_enable 'tableName'
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

- 获取某一行数据

  - 获取某一行所有数据

    ```
    get 'tableName', 'rowkey'
    ```

  - 获取某一行，某个列族的所有信息

    ```
    get 'tableName','rowkey', 'columnFamily'
    ```

  - 获取某一行，某一列的信息

    ```
    get 'tableName', 'rowkey', 'columnFamily:column'
    ```

  - 获取某一行，多个列族的信息

    ```
    get 'tableName', 'rowkey', 'columnFamily1', 'columnFamily2'
    get 'tableName', 'rowkey', {COLUMN => ['columnFamily1', 'columnFamily12']}
    ```

  - 获取某一行，最新 5个版本的数据

    ```
    get 'tableName', 'rowkey', {COLUMN => 'columnFamily', VERSIONS => 5}
    get 'tableName', 'rowkey', {COLUMN => 'columnFamily:column', VERSIONS => 5}
    get 'tableName', 'rowkey', {COLUMN => 'columnFamily:column', VERSIONS => 5, TIMERANGE => [1392368783980, 1392380169184]}
    ```

  - 获取某一行，cell 的值为指定值

    ```
    get 'tableName', 'rowkey', {FILTER => "ValueFilter(=, 'binary:cellValue')"}
    ```

  - 获取某一行，列标识符含有指定的信息

    ```
    get 'tableName', 'rowkey', {FILTER => "(QualifierFilter(=,'substring:columnValue'))"}
    ```

- 扫描表

  - 获取全表数据

    ```
    scan 'tableName'
    ```
    
  - 获取指定列族的数据
  
  ```
    scan 'tableName', {COLUMN => 'columnFamily'}
  ```
  
  - 获取多个列族的数据
  
  ```
    scan 'tableName', {COLUMN => ['columnFamily1', 'columnFamily2']}
  ```
  
  - 获取指定列的数据
  
    ```
  scan 'tableName', {COLUMN => 'columnFamily:column'}
    ```
  
  - 获取多个列的数据
  
    ```
    scan 'tableName', {COLUMN => ['columnFamily1:column', 'columnFamily2:column']}
    ```
  
  - 获取列标识符中包含指定字符的数据
  
    ```
    scan 'tableName', {COLUMN => 'columnFamily', FILTER => "(QualifierFilter(=,'substring:columnValue'))"}
    ```
  
  - 获取 rowkey 范围在 [rowkey001, rowkey003) 内的数据
  
    ```
    scan 'tableName', {STARTROW => 'rowkey001', ENDROW => 'rowkey003'}
    ```
  
  - 查询 rowkey 以 'rk' 字符开头的数据
  
    ```
    scan 'tableName', {FILTER => "PrefixFilter('rk')"}
    ```
  
  - 限制显示的数量
  
    ```
    scan 'tableName', {COLUMN => 'columnFamily', LIMIT => 5}
    ```
  
- 删除数据

  - 删除行

    ```
    delete 'tableName', 'rowkey'
    ```
  
- 删除行中某个列
  
  ```
    delete 'tableName', 'rowkey', 'columnFamily:column'
  ```
  
- 删除表中所有数据
  
  ```
    truncate 'tableName'
  ```

- 查询表的数据量

  ```
  count 'tableName'
  ```

# 退出 Shell

通过 `quit` 退出