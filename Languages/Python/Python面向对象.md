# Python面向对象

## 目录

> * [类和实例](#chapter1)
>
> * [访问限制](#chapter2)

## 类和实例 <a id="chapter1"></a>

类的定义是通过`class`关键字：

```python
class Student(object):
	pass
```

> `class`后面跟着类名，即`Student`，类名通常是大写开头的单词，`(object)`表示该类是从哪个类继承下来的

创建类的实例：

```python
bart = Student()

#给实例绑定属性
bart.name = 'Bart Simpson'
```

在定义类的时候，就给绑定属性：

```python
class Student(object):
    
    def __init__(self,name,score):
        self.name = name
        self.score = score
```

> `__init__`方法的第一个参数永远是`self`，表示创建的实例本身
>
> 有了`__init__`方法，创建实例的时候，就不能传入空的参数，必须传入与`__init__`方法匹配的参数，

## 访问限制 <a id="chapter2"></a>

要内部属性不被外部访问，可以在属性名前加两个下划线`__`，这样这个属性就变成私有变量(private)，只有内部可以访问，外部不能访问：

```python
class Student(object):
    
    def __init__(self,name,score):
        self.__name = name
        self.__score = score
    
    def print_socre(self):
        print('%s:%s' % (self.__name,self.__score))
```

外部代码获取内部属性：

```python
class Student(object):
    ...
    
    def get_name(self):
        return self.__name
    
    def get_score(self):
        return self.__score
```

外部代码修改内部属性：

```python
class Student(object):
    ...
    
    def set_score(self,score):
        self.__score = score
```

> 不能直接访问`__name`是因为Python解释器对外把`__name`变量改成了`_Student__name`，所以通过`_Student_name`仍然可以访问`__name`变量

下划线的使用：

- 变量名类似`__xxx__`，是特殊变量，特殊变量可以直接访问，不是private
- 以一个下划线开头的实例变量名，`_xxx`，这样的变量，虽然可以直接访问，但是不应该直接访问