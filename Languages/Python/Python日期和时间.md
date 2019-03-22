# Python 日期和时间

## 目录

> * [时间元组](#chapter1)
> * [日期格式化符号](#chapter2)
> * [time 模块](#chapter3)

Python 提供一 time 和 calendar 模块处理日期和时间

时间戳是从 1970年1月1日 起经过的时间，单位是秒(s)



## 时间元组 <a id="chapter1"></a>

很多 Python 函数用一个包含9组数字的元组来处理时间：

| 属性     | 取值           | 备注         |
| -------- | -------------- | ------------ |
| tm_year  | 2008           | 年           |
| tm_mon   | 1-12           | 月           |
| tm_mday  | 1-31           | 日           |
| tm_hour  | 0-23           | 小时         |
| tm_min   | 0-59           | 分钟         |
| tm_sec   | 0-61           | 秒           |
| tm_wday  | 0-6（0 是周一) | 一周的第几日 |
| tm_yday  | 1-366          | 一年的第几日 |
| tm_isdst | -1,0,1         | 夏令时       |



## 日期格式化符号 <a id="chapter2"></a>

| 符号 | 备注                         |
| ---- | ---------------------------- |
| %y   | 两位数的年份表示 (00-99)     |
| %Y   | 四位数的年份表示 (000-9999)  |
| %m   | 月份 (01-12)                 |
| %d   | 月内的一天 (0-31)            |
| %H   | 24小时制小时数 (0-23)        |
| %I   | 12小时制小时数 (1-12)        |
| %M   | 分钟数 (00-59)               |
| %S   | 秒 (00-59)                   |
| %a   | 本地简化星期名称             |
| %A   | 本地完整星期名称             |
| %b   | 本地简化的月份名称           |
| %B   | 本地完整的月份名称           |
| %c   | 本地相应的日期表示和时间表示 |
| %j   | 年内的一天 (001-366)         |
| %p   | 本地A.M.或P.M.的等价符       |
| %U   | 一年中的星期数 (00-53)       |
| %w   | 星期 (0-6,0为星期日)         |
| %W   | 一年中的星期数 (00-53)       |
| %x   | 本地相应的日期表示           |
| %X   | 本地相应的时间表示           |
| %Z   | 当前时区的名称               |
| %%   | %号本身                      |



## time 模块 <a id="chapter3"></a>

- time.time()

  获取当前时间戳

  ```python
  ticks = time.time()
  print "当前时间戳为: ",ticks
  ```

  > 输出结果：
  >
  > 当前时间戳为: 1553133289.243858

- time.localtime([secs])

  接受时间戳，返回当地时间下的时间元组

  ```python
  localtime = time.localtime(time.time())
  print "本地时间为: "，localtime
  ```

  > 输出结果：
  >
  > 本地时间为: time.struct_time(tm_year=2019, tm_mon=3, tm_mday=21, tm_hour=10, tm_min=14, tm_sec=54, tm_wday=3, tm_yday=80, tm_isdst=0)

- time.mktime(tupletime)

  接受时间元组，返回时间戳

- time.strftime(fmt,[,tupletime])

  接受时间元组，并返回以可读字符串表示的时间，格式由 format 决定

- time.strptime(str,fmt='Format')

  根据 fmt 的格式把一个时间字符串解析为时间元组

- time.asctime([tupletime])

  接受时间元组，返回一个可读形式为 ‘Thu Mar 21 10:19:28 2019’ 的24个字符的字符串

  ```python
  localtime = time.asctime(time.localtime(time.time()))
  print "本地时间为：",localtime
  ```

  > 输出结果：
  >
  > 本地时间为：Thu Mar 21 10:19:28 2019

- time.clock()

  返回当前的 CPU 时间，以浮点数计算的秒数。可以用来衡量不同程序的耗时

- time.sleep(secs)

  推迟调用线程的运行，secs 为秒数



