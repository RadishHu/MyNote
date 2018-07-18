# Python函数式编程

函数式编程的一个特点是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数

## 高阶函数

一个函数可以接收另一个函数作为参数，这种函数称为高阶函数

函数式编程就是指这种高度抽象的编程范式

- map()

  map()函数接收两个参数，一个是函数，一个是`Iterable`，`map`将传入的函数依次作用到序列的每个元素，并把结果作为新的`Iterator`返回

  示例：

  ```python
  >>> def f(x):
  ...     return x * x
  ...
  >>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
  >>> list(r)
  [1, 4, 9, 16, 25, 36, 49, 64, 81]
  ```

  把list所有数字转换为字符串：

  ```python
  >>> list(map(str, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
  ['1', '2', '3', '4', '5', '6', '7', '8', '9']
  ```

- reduce()

  reduce把一个函数作用在一个序列上，这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做计算：

  ```
  reduce(f,[x1,x2,x3,x4]) = =f(f(f(x1,x2),x3),x4)
  ```

  示例，使用reduce实现对一个序列的求和：

  ```python
  >>> from functools import reduce
  >>> def add(x, y):
  ...     return x + y
  ...
  >>> reduce(add, [1, 3, 5, 7, 9])
  25
  ```

  把序列转化为整数：

  ```python
  >>> from functools import reduce
  >>> def fn(x, y):
  ...     return x * 10 + y
  ...
  >>> reduce(fn, [1, 3, 5, 7, 9])
  13579
  ```

  把str转换为int的函数：

  ```python
  from functools import reduce
  
  DIGITS = {'0':0,'1':1,'2':2,'3':3,'4':4,'5':5,'6':6,'7':7,'8':8,'9':9}
  
  def str2int(s):
      def fn(x,y):
          return x * 10 + y
      def char2num(s):
          return DIGITS[s]
      return reduce(fn,map(char2num,s))
  ```

  使用lambda函数进一步简化：

  ```python
  from functools import reduce
  
  DIGITS = {'0':0,'1':1,'2':2,'3':3,'4':4,'5':5,'6':6,'7':7,'8':8,'9':9}
  
  def char2num(s):
      return DIGITS[S]
  
  def str2int(s):
      return reduce(lambda x,y:x * 10 + y,map(char2num,s))
  ```

- filter()

  filter()函数用于过滤序列

  filter()接收一个函数和一个序列，filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素

  示例，在一个list中，删除偶数，只保留奇数：

  ```python
  def is_odd(n):
      return n % 2 ==1
  
  list(filter(is_odd,[1,2,4,5,6,9,10,15]))
  #结果：[1,5,6,9,15]
  ```

- sorted()

  sorted()函数可以对list进行排序：

  ```python
  >>> sorted([36, 5, -12, 9, -21])
  [-21, -12, 5, 9, 36]
  ```

  sorted()函数可以接受一个key函数来实现自定义的排序，按照绝对值大小进行排序：

  ```python
  >>> sorted([36, 5, -12, 9, -21], key=abs)
  [5, 9, -12, -21, 36]
  ```

  对字符串进行排序：

  ```python
  >>> sorted(['bob', 'about', 'Zoo', 'Credit'])
  ['Credit', 'Zoo', 'about', 'bob']
  ```

  忽略大小写进行排序：

  ```python
  >>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
  ['about', 'bob', 'Credit', 'Zoo']
  ```

  进行反向排序：

  ```python
  >>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
  ['Zoo', 'Credit', 'bob', 'about']
  ```