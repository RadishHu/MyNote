# Python模块

## 目录

> * [包(Package)](#chapter1)

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