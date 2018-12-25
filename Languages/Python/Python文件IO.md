# Python文件I/O

## 目录

> * [打开和关闭文件](#chapter1)
> * [File对象的属性](#chapter2)
> * [文件定位](#chapter3)
> * [文件重命名和删除](#chapter4)
> * [Python中的目录](#chapter5)
> * [File对象](#chapter6)

## 打开和关闭文件 <a id="chapter1"></a>

### open函数<a id="open"></a>

先使用python内置函数open()函数打开一个文件，创建一个file对象

语法：

```python
file = open(file_name,access_mode,buffering)
```

> - file_name：file_name变量是一个包含了要访问的文件的字符串值
> - access_mode：access_mode决定了打开文件的模式：只读、写入、追加等，默认模式为只读(r)
> - buffering：如果buffering的值设为0，就不会有寄存；如果buffering的值设为大于1的整数，表明这个寄存区的缓冲大小；如果设为负值，寄存区的缓冲大小为系统默认

文件打开模式列表：

| 模式 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| t    | 文本模式(默认)                                               |
| b    | 二进制模式                                                   |
| x    | 写模式，新建一个文件，如果该文件已存在则会                   |
| +    | 可读可写                                                     |
| r    | 只读模式，文件的指针会放在文件的开头(默认)                   |
| w    | 只写模式，如果文件已存在，会覆盖文件中的内容；如果文件不存在，创建新文件 |
| a    | 追加写入                                                     |

> 模式组合：
>
> - rb：以二进制格式打开一个只读文件
> - r+：打开一个读写文件，文件指针会放在文件的开头
> - rb+：以二进制格式打开一个读写文件，文件指针会放在文件的开头，一般用于非文本文件
> - wb：以二进制格式打开一个只写文件，会覆盖文件原有的内容
> - w+：打开一个读写文件，从文件开头开始编辑，原有内容会被删除
> - wb+：以二进制格式打开一个读写文件，从文件开头开始编辑
> - ab：以二进制格式打开一个文件，用于追加，文件的指针会放在文件的尾部
> - a+：打开一个读写文件，用于追加
> - ab+：以二进制格式打开一个文件，用于追加

## File对象的属性 <a id = "chapter2"></a>

创建file对象后，就可以获取到文件的各种属性，file对象的属性列表：

| 属性        | 描述                              |
| ----------- | --------------------------------- |
| file.closed | 文件已关闭返回true，否则返回false |
| file.mode   | 返回打开文件的访问模式            |
| file.name   | 返回文件的名称                    |

### close()方法

File对象的close()方法刷新缓冲区里还没写入的信息，并关闭该文件

语法：

```python
file.close()
```

### write()方法

write()方法可将任何字符串写入一个打开的文件，Python字符串可以是二进制数据，而不仅仅是文字

write()方法不会在字符串的结尾添加换行符('\n')

语法：

```python
file.write(string)
```

### read()方法

read()方法从一个打开的文件中读取一个字符串，Python字符串可以是二进制数据

语法：

```python
file.read([count])
```

> count：指要从文件中读取的字节数，该方法从文件的开头开始读入，如果没有传入count，它会尽可能多地读取更多的内容，可能读到文件的末尾

## 文件定位 <a id="chapter3"></a>

tell()方法会告诉你文件内的当前位置，即下一次读写会发生在文件开头多少个字节之后

seek(offset,[from])方法改变文件的当前位置

> offset：指定要移动的字节数
>
> from：指定开始移动字节的位置，设置为0，表示从文件的开头作为参考位置；设置为1，使用当前位置作为参考位置；设置为2，使用文件的末尾作为参考位置

## 文件重命名和删除<a id="chapter4"></a>

Python的os模块提供了文件重命名和删除的方法，使用前需要先导入这个模块

**rename()方法**

文件重命名，语法：

```python
os.rename(current_file_name,new_file_name)
```

**remove()方法**

删除文件，语法：

```python
os.remove(file_name)
```

## Python中的目录<a id="chapter5"></a>

os模块中的方法可以用于删除和更改目录

**mkdir()方法**

在当前目录下创建新的目录，语法：

```python
os.mkdir('newdir')
```

**chdir()方法**

用来改变当前的目录，语法：

```python
os.chdir('newdir')
```

**getcwd方法**

显示当前的工作目录，语法：

```
os.getcwd()
```

**rmdir()方法**

删除目录，删除目录前，它里面的所有内容应该先被删除，语法：

```python
os.rmdir('dirname')
```

## File对象<a id="chapter6"></a>

File对象提供了操作文件的一系列方法

File对象使用open函数创建，open函数的使用可以参照[open函数](#open)

File对象的常用函数：

**file.close()**

关闭文件，关闭后不能再进行读写操作

**file.flush()**

刷新文件内部缓冲，直接把内部缓冲的数据立刻写入文件

**file.next()**

返回文件下一行

**file.read([size])**

从文件读取指定的字节数，如果未给定或为负则读取所有

**file.readline([size])**

读取整行，包括"\n"字符

**file.readlines([sizeint])**

读取所有行并返回列表，若给定sizeint>0，则是设置一次读多少字节

**file.seek(offset,[whence])**

设置文件当前位置

**file.tell()**

返回文件当前位置

**file.truncate([size])**

截取文件，截取的字节通过size指定，默认为当前文件位置

**file.write(str)**

将字符串写入文件，返回的是写入的字符长度

**file.writelines(sequence)**

向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符