# Python连接MySQL

## 目录

> * [Linux安装pymysql](#chapter1)
> * [连接MySQL](#chapter2)
> * [执行SQL语句](#chapter3)

这里使用pymysql模块连接mysql，在使用pymysql之前需要先安装模块，可以通过pip安装，也可以离线安装

## Linux安装pymysql <a id="chapter1"></a>

- 下载[pymysql](https://pypi.org/)，并上传到服务器

- 解压缩

  ```shell
  tar zxvf PyMySQL-0.9.3.tar.gz
  ```

- 进入解压目录

  ```shell
  cd PyMySQL-0.9.3
  ```

- 安装pymysql(需要root权限)

  ```shell
  sudo python setup.py install
  ```

## 连接MySQL<a id="chapter2"></a>

引入pymysql模块

```
#! /usr/bin/python
# -*-coding:utf-8-*-

import pymysql
```

创建connec、cursort对象

```python
connect = pymysql.connect(
    host='127.0.0.1',
    port=3306,
    user='username',
    passwd='password',
    db='database',
    charset='utf8mb4'
)

# 创建游标
cursor = connect.cursor()

# 关闭cursor、connect
cursor.close()
connect.close()
```

## 执行SQL语句<a id="chapter3"></a>

建表

```python
sql_drop = "DROP TABLE IF EXISTS test_table;"
cursor.execute(sql_drop)
connect.commit()

sql_create = '''
CREATE TABLE IF NOT EXISTS test_table (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(255) NOT NULL COMMENT ,
`age` int(11) DEFAULT NULL
PRIMARY KEY (`id`));
'''
cursor.execute(sql_create)
connect.commit()
```

插入数据

```python
sql = "INSERT INTO test_table (id,name,age) VALUES (%d,'%s','%d')"
data = (1,"LiLi",20)
cursor.execute(sql % data)
connect.commit()
```

修改数据

```python
sql = "UPDATE test_table SET age = '%d' WHERE name = %s"
data = (21,"LiLi")
cursor.execute(sql % data)
connect.commit()
```

查询数据

```python
sql = "SELECT id,name,age FROM test_table WHERE name = %d"
data = ("LiLi",)
cursor.execute(sql % data)
for row in cursor.fetchall():
    print(type(row))
    print("id:%d\tname:%s\tage:%d" % row)
```

删除数据

```python
sql = "DELETE FROM test_table WHERE name = %d"
data = ("LiLi",)
cursor.execute(sql % data)
connect.commit()
```

事务处理

```python
sql_1 = "INSERT INTO test_table (id,name,age) VALUES (1,'LiLi',20)"
sql_2 = "INSERT INTO test_table (id,name,age) VALUES (2,'MingMing',21)"
sql_3 = "INSERT INTO test_table (id,name,age) VALUES (3,'QiQi',25)"

try:
    cursor.execute(sql_1)
    cursor.execute(sql_2)
    cursor.execute(sql_3)
except Exception as e:
    connect.rollback() # 事务回滚
    print('事务处理失败:',e)
else:
    connect.commit() # 事务提交
    print('事务处理成功')

cursor.close()
connect.close()
```

