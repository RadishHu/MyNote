# Python高级特性

## 切片

切片可以用于从有序集合中截取部分数据

- List

  ```python
  >>> L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
  ```

  取前三个元素：

  ```python
  >>> L[0:3]
  ['Michael', 'Sarah', 'Tracy']
  ```

  从索引0开始，直到索引3为止，但不包括索引3

  第一个索引缺省值是0，第二个索引缺省值为集合的长度：

  ```python
  >>> L[:3]
  ['Michael', 'Sarah', 'Tracy']
  
  >>> L[1:]
  ['Sarah', 'Tracy', 'Bob', 'Jack']
  ```

  进行间隔截取：

  ```python
  >>> L = list(range(100))
  >>> L
  [0, 1, 2, 3, ..., 99]
  
  #前10个数，每两个取一个
  >>> L[:10:2]
  [0, 2, 4, 6, 8]
  
  #所有数，每5个取一个
  >>> L[::5]
  [0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]
  
  #原样复制一个list
  >>> L[:]
  [0, 1, 2, 3, ..., 99]
  ```

- tuple

  tuple也是一种list，唯一区别是tuple是不可变的，因此tuple也可以用切片操作：

  ```python
  >>> (0, 1, 2, 3, 4, 5)[:3]
  (0, 1, 2)
  ```

- 字符串

  字符串也可以看成一个list，每个元素是一个字符，因此字符串也可以用切片操作：

  ```python
  >>> 'ABCDEFG'[:3]
  'ABC'
  >>> 'ABCDEFG'[::2]
  'ACEG'
  ```

## 迭代

- dict迭代

  ```python
  >>> d = {'a': 1, 'b': 2, 'c': 3}
  >>> for key in d:
  ...     print(key)
  ...
  a
  c
  b
  ```

  dict默认迭代的是 key，如果要迭代value，可以用`for value in d.values()`。如果同时迭代key和value，可以用`for k,v in d.items()`

- 字符串迭代

  ```python
  >>> for ch in 'ABC':
  ...     print(ch)
  ...
  A
  B
  C
  ```

- list

  list迭代索引-元素对，使用python内置的`enumerate`函数：

  ```python
  >>> for i, value in enumerate(['A', 'B', 'C']):
  ...     print(i, value)
  ...
  0 A
  1 B
  2 C
  ```

- 判断一个对象是否为可迭代对象：

  ```python
  >>> from collections import Iterable
  >>> isinstance('abc', Iterable) # str是否可迭代
  True
  >>> isinstance([1,2,3], Iterable) # list是否可迭代
  True
  >>> isinstance(123, Iterable) # 整数是否可迭代
  False
  ```

## 列表生成式

- 生成一个list

  ```python
  >>> list(range(1, 11))
  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  ```

- 生成`[1x1, 2x2, 3x3, ..., 10x10] `，使用for循环

  方法一：

  ```python
  >>> L = []
  >>> for x in range(1, 11):
  ...    L.append(x * x)
  ...
  >>> L
  [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
  ```

  方法二：

  ```python
  >>> [x * x for x in range(1, 11)]
  [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
  ```

  for循环后加判断：

  ```python
  >>> [x * x for x in range(1, 11) if x % 2 == 0]
  [4, 16, 36, 64, 100]
  ```

  使用两层循环，生成全排列：

  ```python
  >>> [m + n for m in 'ABC' for n in 'XYZ']
  ['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
  ```

  同时使用两个变量：

  ```python
  >>> d = {'x': 'A', 'y': 'B', 'z': 'C' }
  >>> for k, v in d.items():
  ...     print(k, '=', v)
  ...
  y = B
  x = A
  z = C
  ```

- 获取目录下所有文件和目录名

  ```python
  >>> import os # 导入os模块，模块的概念后面讲到
  >>> [d for d in os.listdir('.')] # os.listdir可以列出文件和目录
  ['.emacs.d', '.ssh', '.Trash', 'Adlm', 'Applications', 'Desktop', 'Documents', 'Downloads', 'Library', 'Movies', 'Music', 'Pictures', 'Public', 'VirtualBox VMs', 'Workspace', 'XCode']
  ```

- 把list中所有字符串变变成小写

  ```python
  >>> L = ['Hello', 'World', 'IBM', 'Apple']
  >>> [s.lower() for s in L]
  ['hello', 'world', 'ibm', 'apple']
  ```

## 生成器

列表元素按照某种算法推算出来，在循环过程中不断推算出后续的元素，这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：generator

- 创建generator，把列表生成式的`[]`换成`()`：

  ```python
  >>> L = [x * x for x in range(10)]
  >>> L
  [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
  >>> g = (x * x for x in range(10))
  >>> g
  <generator object <genexpr> at 0x1022ef630
  ```

- 打印generator中的值

  通过next()函数一个一个打印：

  ```python
  >>> next(g)
  0
  >>> next(g)
  1
  ...
  >>> next(g)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  StopIteration
  ```

  generator保存的是算法，每次调用next(g)，就计算出g下一个元素的值，直到计算到最后一个元素，没有更多元素的时候，抛出`StopIteration`的错误

  通过for循环，generator是迭代对象：

  ```python
  >>> g = (x * x for x in range(10))
  >>> for n in g:
  ...     print(n)
  ```