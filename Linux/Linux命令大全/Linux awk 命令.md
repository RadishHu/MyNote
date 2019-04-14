# Linux awk 命令

## 目录

> - [语法](#chapter1)

awk 是一种处理文本文件的语言，是一个强大的文本分析工具

awk 是一个行处理器，在处理庞大文件时不会出现内存溢出或处理缓慢的问题



## 语法 <a id="chapter1"></a>

```shell
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```

> 选项参数：
>
> - -F fs / --field-separator fs
>
>   指定输入文件分隔符，fs 是一个字符串或一个正则表达式
>
> - -v var=value / --asign var=value
>
>   赋值给一个用户自定义变量
>
> - -f scriptfile / --file scriptfile
>
>   指定脚本文件
>
> - -W help / --help
>
>   打印全部 awk 选项和每个选项的简短说明



## 基本用法 <a id="chapter2"></a>

test.log 文本内容如下：

```
1 linux awk
2 awk test
3,test,awk
```

用法一：



## 参考

[【菜鸟教程】《Linux awk 命令》](<http://www.runoob.com/linux/linux-comm-awk.html>)

[awk 学习](<http://blog.chinaunix.net/uid-23302288-id-3785105.html>)



