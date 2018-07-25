# Python模块

## 目录

> * [包(Package)](#chapter1)
> * [使用模块](#chapter2)
> * [作用域](#chapter3)

Python中，一个`.py`文件就称之为一个模块(Module)

使用模块可以避免函数名和变量名冲突，相同名字的函数和变量完全可以分别存在不同的模块中

## 包(Package) <a id="chapter1"></a>

Python中引入按目录来组织模块的方法，称为包(Package)

通过包可以避免模块名冲突：  

```
mycompany
|—— web
|	 |—— _init_.py
|	 |—— utils.py
|	 |—— www.py
|—— _init_.py
|—— abc.py
|—— xyz.py
```

> abc.py模块的名字就成了`mycompany.abc`，www.py模块的名字就成了`mycompany.web.www`
>
> 每个包目录下都会有一个`_init_py`的文件，这个文件必须存在，否则，Python就把这个目录当成普通目录，而不是一个包
>
> `_init_py`可以是空文件，也可以是Python代码，因为`_init_py`本身就是一个模块，它的模块名为`mycompany`

## 使用模块 <a id="chpater2"></a>

Python内置了许多模块，这些模块都可以使用

使用内建的`sys`模块，编写一个`hello`模块：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

' a test module '

__author__ = 'YuTou'

import sys

def test():
    args = sys.argv
    if len(args)==1:
        print('Hello, world!')
    elif len(args)==2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')

if __name__=='__main__':
    test()
```

> * 第一行和第二行是标准注释，第一行注释让`hello.py`文件直接可以Unix/Linux/Mac上运行。第二行注释表示.py文件本身使用标准UTF-8编码
>
> * 第四行是模块的文档注释，任何模块的第一行字符串都被视为模块的文档注释
>
> * 第六行使用`_author_`变量把作者写入
>
> * 使用`sys`模块，要先导入该模块：
>
>   ```
>   import sys
>   ```
>
>   导入`sys`模块后，就会有一个`sys`变量指向该模块，利用`sys`变量就可以访问`sys`模块的所有功能

## 作用域 <a id="chapter3"></a>

正常的函数和变量名是公开的(public)，可以直接引用，比如`abc`、`x123`、`PI`等

类似`_xxx`这样的函数或变量是非公开的(private)，不应该被直接引用，但是并不是说“不能”被直接引用

类似`_xxx_`这样的特殊变量，可以被直接引用，但是有特殊用途，比如`_author`，`_name_`就是特殊变量，`hello`模块定义的文档注释可以 用特殊变量`_doc_`访问

## 安装第三方模块

- pip

  通过包管理工机具pip可以安装第三方模块

  可以通过Python官方的[pypi.python.org](https://pypi.python.org/) 网站查询第三方库的名称，然后通过pip安装。安装Pillow：

  ```python
  pip install Pillow
  ```

- Anaconda

  使用pip一个个安装第三方模块费时费力，还要考虑兼容性。因此可以使用Anaconda,它是一个基于Python的数据处理和科学计算平台，内置了许多常用的第三方库

### 模块搜索路径

当加载一个模块时，Python会在指定路径下搜索对应的.py文件，搜索路径存放在`sys`模块的`path`变量中：

```python
import sys
sys.path
```

添加搜索路径的方法：

- 直接修改`sys.path`：

  ```python
  import sys
  sys.path.append('/my/modle')
  ```

- 设置环境变量`PTTHONPATH`，该环境变量的内容会被自动添加到搜索路径中

