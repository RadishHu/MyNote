# Python函数

## 内置函数

- abs，求绝对值

  ```python
  >>> abs(100)
  100
  ```

- 获取最大值

  ```python
  >>> max(2, 3, 1, -5)
  3
  ```

- 数据类型转换

  ```python
  >>> int('123')
  123
  >>> int(12.34)
  12
  >>> float('12.34')
  12.34
  >>> str(1.23)
  '1.23'
  >>> str(100)
  '100'
  >>> bool(1)
  True
  >>> bool('')
  False
  ```

## 定义函数

- 函数定义格式

  ```python
  def 函数名(参数)：
  	函数体
      return
  ```

  示例：

  ```python
  def my_abs(x):
      if x >= 0:
          return x
      else:
          return -x
  ```

  如果没有return语句，函数执行执行完毕后会返回None

- 定义空函数

  ```python
  def nop():
      pass
  ```

  pass用来作为占位符，缺少pass，代码运行会有语法错误

- 参数检查

  调用函数，如果参数个数不对，Python解释器会自动检查出来，并抛出`TypeError`

  如果参数类型不对，Python解释器无法检查出来

  使用内置函数`isinstance()`检查数据类型：

  ```python
  def my_abs(x):
      if not isinstance(x, (int, float)):
          raise TypeError('bad operand type')
      if x >= 0:
          return x
      else:
          return -x
  ```

- 定义返回多个值的函数：

  ```python
  import math
  
  def move(x, y, step, angle=0):
      nx = x + step * math.cos(angle)
      ny = y - step * math.sin(angle)
      return nx, ny
  ```

  这样返回的值是一个tuple：

  ```python
  >>> r = move(100, 100, 60, math.pi / 6)
  >>> print(r)
  (151.96152422706632, 70.0)
  ```

  返回的tuple可以用多个变量进行接收，按位置赋给对对应的值：

  ```python
  >>> x, y = move(100, 100, 60, math.pi / 6)
  >>> print(x, y)
  151.96152422706632 70.0
  ```


## 函数参数

- 默认参数

  为函数的参数设置一个默认值

  ```python
  def power(x, n=2):
      s = 1
      while n > 0:
          n = n - 1
          s = s * x
      return s
  ```

  当调用power(5)时，相当于调用power(5,2)

  设置默认参数的注意事项：

  - 必选参数在前，默认参数在后
  - 默认参数必须指向不可变对象

- 可变参数

  ```python
  def calc(*numbers):
      sum = 0
      for n in numbers:
          sum = sum + n * n
      return sum
  ```

  可变参数numbers接收的是一个tuple，调用函数时可以传入任意个参数，包括0个参数

  可以把list或tuple直接传给可变参数：

  ```python
  >>> nums = [1, 2, 3]
  >>> calc(*nums)
  14
  ```

- 关键字参数

  关键字参数允许传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict

  示例：

  ```python
  def person(name, age, **kw):
      print('name:', name, 'age:', age, 'other:', kw）
  
  #传入任意个数的关键字参数
   >>> person('Bob', 35, city='Beijing')
  name: Bob age: 35 other: {'city': 'Beijing'}        
  ```

  可以先组装一个dict，然后把该dict作为参数传入：

  ```python
  >>> extra = {'city': 'Beijing', 'job': 'Engineer'}
  >>> person('Jack', 24, **extra)
  name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
  ```

  