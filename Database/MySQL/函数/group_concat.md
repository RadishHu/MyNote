# group_concat函数

- 语法：

  ```sql
  group_concat([DISTINCT] 要连接的字段 [order by 排序字段 ASC/DESC] [Separator '分隔符'])
  ```

- 示例

  基础查询：

  ```sql
  select * from test; 
  
  +------+------+
  | id| name |
  +------+------+
  |1 | 10|
  |1 | 20|
  |1 | 20|
  |2 | 20|
  |3 | 200 |
  |3 | 500 |
  +------+------+
  ```

  以id分组，把name字段的值打印在一行，逗号分隔(默认)：

  ```sql
  select id,group_concat(name) from test group by id;
  
  +------+--------------------+
  | id| group_concat(name) |
  +------+--------------------+
  |1 | 10,20,20|
  |2 | 20 |
  |3 | 200,500|
  +------+--------------------+
  3 rows in set (0.00 sec)
  ```

  以id分组，把name字段的值打印在一行，分号分隔：

  ```sql
  select id,group_concat(name separator ';') from test group by id; 
  
  +------+----------------------------------+
  | id| group_concat(name separator ';') |
  +------+----------------------------------+
  |1 | 10;20;20 |
  |2 | 20|
  |3 | 200;500 |
  +------+----------------------------------+
  ```

  以id分组，把冗余的name字段值去除：

  ```sql
  select id,group_concat(distinct name) from test group by id; 
  
  +------+-----------------------------+
  | id| group_concat(distinct name) |
  +------+-----------------------------+
  |1 | 10,20|
  |2 | 20 |
  |3 | 200,500 |
  +------+-----------------------------+
  ```

  以id分组，把name字段值打印在一行，逗号分隔，以name排倒序：

  ```sql
  select id,group_concat(name order by name desc) from test group by id;  
  
  +------+---------------------------------------+
  | id| group_concat(name order by name desc) |
  +------+---------------------------------------+
  |1 | 20,20,10 |
  |2 | 20|
  |3 | 500,200|
  +------+---------------------------------------+
  ```

  

