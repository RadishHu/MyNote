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

  