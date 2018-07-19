# Python函数式编程

函数式编程的一个特点是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数

## 目录

> * [高阶函数](#chapter1)
> * [返回函数](#chapter2)
> * [匿名函数](#chapter3)
> * [偏函数](#chapter4)

## 高阶函数 <a id="chapter1"></a> 

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

## 返回函数 <a id="chapter2"></a>

高阶函数除了可以接受函数作为参数外，还可以把函数作为结果返回

示例：

通常情况下的求和函数定义：

```python
def calc_sum(*args):
    ax = 0
    for n in args:
        ax = ax + n
    return ax
```

返回求和函数：

```python
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum
```

调用`lazy_sum()`时，返回的并不是求和结果，而是求和函数：

```python
>>> f = lazy_sum(1, 3, 5, 7, 9)
>>> f
<function lazy_sum.<locals>.sum at 0x101c6ed90>
```

调用`f`时，才真正计算求和的结果：

```python
>>> f()
25
```

> * 函数`lazy_sum`中定义了函数`sum`,并且内部函数`sum`引用外部函数`lazy_sum`的参数和局部变量，当`lazy_sum`返回函数`sum`时，相关参数和变量都保存在返回函数中，这样就产生了闭包
> * 闭包是一个函数返回值依赖于声明在函数外部的一个或多个变量
> * 返回函数并没有立刻执行，而是直接调用`f()`才执行

## 匿名函数 <a id="chapter3"></a>

匿名函数，当需要传入函数时，有时不需要显示地定义函数，而是直接传入匿名函数

匿名函数的限制：只能有一个表达式，不用写return，返回值就是该表达式的结果

示例，使用map()函数，计算f(x)=x*x:

```python
>>> list(map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

> 关键字`lambda`表示匿名函数，冒号前面的`x`表示函数参数

## 偏函数 <a id="chapter4"></a>

偏函数(Partial function)是Python的`funtools`模块的提供的一个功能

`int()`函数提供了`base`参数，默认值为10，用于指定转换的进制：

```python
>>> int('12345', base=8)
5349
>>> int('12345', 16)
74565
```

如果要进行大量的二进制字符串转换，每次传入`int(x,base=2)`很繁琐，可以定义一个`int(2)`函数，默认把`base=2`传入：

```python
def int2(x, base=2):
    return int(x, base)

>>> int2('1000000')
64
```

`functools.partial`可以帮助我们创建一个偏函数，不需要自己定义`int2()`：

```python
>>> import functools
>>> int2 = functools.partial(int, base=2)
>>> int2('1000000')
64
```

