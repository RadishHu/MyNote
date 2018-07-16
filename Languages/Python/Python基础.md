# Python基础

## 数据类型

- 整数

  Python可以处理任意大小的整数，包括负数

  十六进制整数，以0x作为前缀

- 浮点数

- 字符串

  字符串时以单引号‘或双引号“括起来的任意文本

- 布尔值

  True、False

- 空值

  None

- 变量

  Python在创建变量时不用指定类型

- 常量

  用全部大写的变量名表示常量

## 运算符

- 除法

  /除法计算结果是浮点数

  //除法计算结果是浮点数，取结果的整数部分

- 逻辑运算符

  and、or、not

## Python脚本文件的开头

```python
#! /usr/bin/env python3
# -*- coding:utf-8 -*-
```

第一行指定解释器

第二行告诉解释器使用utf-8编码读取文件

## 格式化输出

```python
>>> 'Hello, %s' % 'world'
'Hello, world'
>>> 'Hi, %s, you have $%d.' % ('Michael', 1000000)
'Hi, Michael, you have $1000000.'
```

有几个`%?`占位符，后面就要跟几个变量或值，并且顺序要对应，如果只有一个占位符，括号可以省略

常见占位符：

| 占位符 | 替换内容     |
| ------ | ------------ |
| %d     | 整数         |
| %f     | 浮点数       |
| %s     | 字符串       |
| %x     | 十六进制整数 |

如果要输出的字符串中包含了%，可以进行转义，用`%%`表示一个%：

```python
>>> 'growth rate: %d %%' % 7
```

## List

- 定义list

  ```python
  >>> classmates = ['Michael', 'Bob', 'Tracy']
  ```

- 获取list元素个数

  ```python
  >>> len(classmates)
  3
  ```

- 通过索引获取元素

  ```python
  >>> classmates[0]
  'Michael'
  ```

  当索引超出范围时，Python会报`IndexError`错误

  获取倒数第N个元素元素：

  ```python
  # 获取最后一个元素
  >>> classmates[-1]
  >>> classmates[-2]
  ```

- 追加元素到末尾

  ```python
  >>> classmates.append('Adam')
  ```

  插入元素到指定位置：

  ```python
  >>> classmates.insert(1, 'Jack')
  ```

- 删除元素

  删除list末尾的元素：

  ```python
  >>> classmates.pop()
  ```

  删除指定位置的元素：

  ```python
  >>> classmates.pop(1)
  ```

- 修改指定位置的元素

  ```python
  >>> classmates[1] = 'Sarah'
  ```

## Tuple

元组，一种有序列表，tuple和list非常相似，但是tuple一旦初始化就不能修改

- 定义tuple

  ```python
  >>> classmates = ('Michael', 'Bob', 'Tracy')
  ```

- 定义一个空的tuple

  ```python
  >>> t = ()
  ```

- 定义一个只有一个元素的tuple

  ```python
  >>> t = (1,)
  ```

  加一个逗号，以免和数学意义上的括号混淆

## dict

dict, dictionary, 字典，在其它语言中也称为map，使用键-值(key-value)存储

- 创建dict

  ```python
  >>> d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
  >>> d['Michael']
  95
  ```

- 判断key是否存在

  - 通过in判断

    ```python
    >>> 'Thomas' in d
    False
    ```

  - 通过get()方法，如果key不存在，就会返回None，或者自己指定的value

    ```python
    >>> d.get('Thomas')
    >>> d.get('Thomas', -1)
    -1
    ```

- 删除key

  ```python
  >>> d.pop('Bob')
  75
  ```

## Set

- 创建set，需要提供一个list作为输入

  ```python
  >>> s = set([1, 2, 3])
  >>> s
  {1, 2, 3}
  ```

- 添加元素

  ```python
  >>> s.add(4)
  ```

- 删除元素

  ```python
  >>> s.remove(4)
  ```

- set做交集、并集操作

  ```python
  >>> s1 = set([1, 2, 3])
  >>> s2 = set([2, 3, 4])
  >>> s1 & s2
  {2, 3}
  >>> s1 | s2
  {1, 2, 3, 4}
  ```

## 条件判断

- if语句

  ```python
  age = 20
  if age >= 18:
      print('your age is', age)
      print('adult')
  ```

- if-else语句

  ```python
  age = 3
  if age >= 18:
      print('adult')
  elif age >= 6:
      print('teenager')
  else:
      print('kid')
  ```

- elif语句

  ```python
  if <条件判断1>:
      <执行1>
  elif <条件判断2>:
      <执行2>
  elif <条件判断3>:
      <执行3>
  else:
      <执行4>
  ```

## 循环

- for-in

  ```python
  names = ['Michael', 'Bob', 'Tracy']
  for name in names:
      print(name)
  ```

- while

  ```python
  sum = 0
  n = 99
  while n > 0:
      sum = sum + n
      n = n - 2
  print(sum)
  ```

- range()函数

  生成一个整数序列

  ```python
  >>> list(range(5))
  [0, 1, 2, 3, 4]
  ```

- break

  提前退出循环

  ```python
  n = 1
  while n <= 100:
      if n > 10: # 当n = 11时，条件满足，执行break语句
          break # break语句会结束当前循环
      print(n)
      n = n + 1
  print('END')
  ```

- continue

  跳出当前这次循环

  ```python
  n = 0
  while n < 10:
      n = n + 1
      if n % 2 == 0: # 如果n是偶数，执行continue语句
          continue # continue语句会直接继续下一轮循环，后续的print()语句不会执行
      print(n)
  ```

