# WordCount

## 目录

> - [Python](#chapter1)

## Python <a id="chapter1"></a>

```python
#! /usr/bin/python
# -*-coding:utf-8-*-

#--------------------------
# 通过Python实现WordCount
#--------------------------

# 读取文件
filePath = "G:\Test\WordCount\words.txt"
file = open(filePath,"r")
data = file.read()
file.close()

# 文本前期处理
str_list = data.replace('\n',' ').lower().split(' ')
count_dict = {}

# 如果单词表里有该单词则加1，否则添加入字典
for str in str_list:
    if str in count_dict.keys():
        count_dict[str] = count_dict[str] + 1
    else:
        count_dict[str] = 1

# 输出结果
for key in count_dict.keys():
    print("%s:%d"%(key,count_dict[key]))

```

数据格式：

```
spark hive mapred
sqoop azkaban hdfs
spark hadoop hive
yarn sqoop mapred
```

