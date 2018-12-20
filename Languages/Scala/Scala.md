# Scala基础

## Scala基础语法

Scala与Java的最大区别是：Scala语句末尾的分号`；`是可选的

基础概念：

- 对象，对象有属性和行为
- 类，类是对象的抽象
- 方法，方法描述基本的行为，一个类可以包含多个方法
- 字段，每个对象都有它唯一的实例变量集合，即字段。对象的属性通过给字段赋值来创建

### 基本语法

- 区分下小写

- 类名

  对于所有的类名，第一个字母都要大写。多个单词的话，每个单词的第一个字母要大写

  ```scala
  class MyFirstScalaClass
  ```

- 方法名称

  所有方法名称的第一个字母用小写。多个单词，除了第一个单词，其它单词首字母大写

  ```scala
  def myMethodName()
  ```

- 程序文件名

  程序文件的名称应该与对象名称完全匹配，若不匹配，程序无法编译

  “HelloWorld“为对象名称，那文件名应保存为"HelloWorld.scala"

- def main(args:Array[String])

  Scala程序从main()方法开始处理，这个每个Scala程序的强制程序入口

### 标识符

Scala可以使用两种形式的标识符：字符数字和符号：

- 字符数字

  字符数字使用字母或下划线开头，后面可以接字母或数字

  符号`$`在Scala中也看作为字母，以`$`开始的标识符为保留的Scala编译器产生的标志符使用，应用程序应该避免使用$开始的标识符

### Scala注释

```scala
object HelloWorld {
   /* 这是一个 Scala 程序
    * 这是一行注释
    * 这里演示了多行注释
    */
   def main(args: Array[String]) {
      // 输出 Hello World
      // 这是一个单行注释
      println("Hello, world!") 
   }
}
```

### 空格和空行

一行中只有空格或者带有注释，Scala会认为其为空行，会忽略它

## Scala包

### 定义包

Scala使用package关键字定义包，在Scala将代码定义到某个包中有两种方式：

- 第一种和java一样，在文件头定义包名：

  ```scala
  package com.runoob
  class HelloWorld
  ```

- 第二种方法类似c#：

  ```scala
  package com.runoob {
    class HelloWorld 
  }
  ```

### 引用

Scala使用import关键字引用包

```scala
import java.awt.Color  // 引入Color
 
import java.awt._  // 引入包内所有成员
 
def handler(evt: event.ActionEvent) { // java.awt.event.ActionEvent
  ...  // 因为引入了java.awt，所以可以省去前面的部分
}
```

import语句可以出现在任何地方，而不仅仅是在文件顶部。import的效果从开始延伸到语句块的结束

如果要引入包中的几个成员，可以使用selector(选择器)：

```scala
import java.awt.{Color, Font}
 
// 重命名成员
import java.util.{HashMap => JavaHashMap}
 
// 隐藏成员
import java.util.{HashMap => _, _} // 引入了util包的所有成员，但是HashMap被隐藏了
```

默认情况下，Scala总会引入java.lang._  scala._  和 Predef._

## 数据类型

Scala支持的数据类型：

- Byte
- Short
- Int
- Long
- Float
- Double
- Char
- String
- Boolean
- Unit
- Null
- Nothing
- Any
- AnyRef

这些数据类型都是对象，scala没有java中的原生类型

### 基础字面量

- 整形字面量

  整型字面量用于 Int 类型，如果表示 Long，可以在数字后面添加 L 或者小写 l 作为后缀。

- 浮点型字面量

  如果浮点数后面有f或者F后缀时，表示这是一个Float类型，否则就是一个Double类型的。

- 布尔型字面量

  布尔型字面量有 true 和 false。

- 符号字面量

  符号字面量被写成： **'<标识符>** ，这里 **<标识符>** 可以是任何字母或数字的标识（注意：不能以数字开头）。这种字面量被映射成预定义类scala.Symbol的实例。

  如：符号字面量 'x是表达式 scala.Symbol("x") 的简写，符号字面量定义如下：

  ```scala
  package scala
  final case class Symbol private (name: String) {
     override def toString: String = "'" + name
  }
  ```

- 字符字面量

  在 Scala 字符变量使用单引号 ' 来定义，如下：

  ```scala
  'a' 
  '\u0041'
  '\n'
  '\t'
  ```

  > `\`表示转义字符

- 字符串字面量

  Scala 字符串变量使用双引号 " 来定义：

  ```scala
  "Hello,\nWorld!"
  "菜鸟教程官网：www.runoob.com"
  ```

- 多行字符串的表示

  多行字符串用三个双引号来表示分隔符，格式为：""" ... """

  ```scala
  val foo = """菜鸟教程
  www.runoob.com
  www.w3cschool.cc
  www.runnoob.com
  以上三个地址都能访问"""
  ```

- Null值

  空值是 scala.Null 类型

## Scala变量

### 变量声明

Scala中，使用关键词`var`声明变量，使用关键词`val`声明常量

```scala
var myVar : String = "Foo"
val myVal : String = "Foo"
```

> 变量声明一定需要初始值，否则会报错
>
> 常量定义后不能修改，如果尝试修改常量，程序会在编译时报错

### 变量类型声明

变量的类型在变量名之后，等号之前声明

```scala
var VariableName : DataType = Initial Value

或

val VariableName : DataType = Initial Value
```

### 多个变量声明

Scala支持多个变量声明

```scala
val xmax, ymax = 100  // xmax, ymax都声明为100
```

## 访问修饰符

Scala访问修饰符有：private、protected、public

> 如果没有指定访问修饰符，默认情况下，Scala对象的访问级别是public

### Private

用private关键字修饰，带由此标记的成员仅在包含了成员定义的类或对象内部可见，也适用于内部类

```scala
class Outer{
    class Inner{
    private def f(){println("f")}
    class InnerMost{
        f() // 正确
        }
    }
    (new Inner).f() //错误
}
```

### Protected

用protected修饰，只允许保护成员在定义了该成员的类的子类中被访问

```scala
package p{
class Super{
    protected def f() {println("f")}
    }
    class Sub extends Super{
        f()
    }
    class Other{
        (new Super).f() //错误
    }
}
```

> Sub 类对 f 的访问没有问题，因为 f 在 Super 中被声明为 protected，而 Sub 是 Super 的子类。相反，Other 对 f 的访问不被允许，因为 other 没有继承自 Super。

### Public

public修饰的成员，可以在任何地方被访问

```scala
class Outer {
   class Inner {
      def f() { println("f") }
      class InnerMost {
         f() // 正确
      }
   }
   (new Inner).f() // 正确因为 f() 是 public
}
```

### 作用域保护

Scala中，访问修饰符可以通过使用限定词强调：

```scala
private[x] 
protected[x]
```

> 这里的x指代某个所属的包、类或单例对象。如果写成private[x],读作"这个成员除了对[…]中的类或[…]中的包中的类及它们的伴生对像可见外，对其它所有类都是private。
>
> 这种技巧在横跨了若干包的大型项目中非常有用，它允许你定义一些在你项目的若干子包中可见但对于项目外部的客户却始终不可见的东西。

## Scala IF语句

### if语句

语法：

```scala
if(布尔表达式)
{
   // 如果布尔表达式为 true 则执行该语句块
}
```

示例：

```scala
object Test {
   def main(args: Array[String]) {
      var x = 10;

      if( x < 20 ){
         println("x < 20");
      }
   }
}
```

### if...else语句

语法：

```scala
if(布尔表达式){
   // 如果布尔表达式为 true 则执行该语句块
}else{
   // 如果布尔表达式为 false 则执行该语句块
}
```

### if...else if...else语句

语法：

```scala
if(布尔表达式 1){
   // 如果布尔表达式 1 为 true 则执行该语句块
}else if(布尔表达式 2){
   // 如果布尔表达式 2 为 true 则执行该语句块
}else if(布尔表达式 3){
   // 如果布尔表达式 3 为 true 则执行该语句块
}else {
   // 如果以上条件都为 false 执行该语句块
}
```

### if...else嵌套语句

语法：

```scala
if(布尔表达式 1){
   // 如果布尔表达式 1 为 true 则执行该语句块
   if(布尔表达式 2){
      // 如果布尔表达式 2 为 true 则执行该语句块
   }
}
```

## Scala循环

Scala支持的循环类型：

- while循环
- do...while循环
- for循环

循环控制语句：

Scala不支持break或continue语句

## Scala方法和函数

Scala有方法和函数，但是两者区别比较小：

Scala方法是类的一部分，而函数是一个对象可以赋值给一个变量。换句话就是类中的定义的函数就是方法

Scala中的函数是一个完整的对象，Scala中的函数继承了Trait的类的对象

Scala中使用val语句定义函数，使用def语句定义方法

```scala
class Test{
  def m(x: Int) = x + 3
  val f = (x: Int) => x + 3
}
```

### 方法声明

方法声明格式：

```scala
def functionName ([参数列表]) : [return type]
```

> 如果不写等号和方法体，那方法会被隐式声明为抽象，包含它的类于是也是一个抽象类型

### 方法定义

定义格式：

```scala
def functionName ([参数列表]) : [return type] = {
   function body
   return [expr]
}
```

> return type可以是任意合法的Scala数据类型，参数列表中的参数可以使用都好分隔

示例：

```scala
object add{
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
```

如果方法没有返回值，可以分会为Unit：

```scala
object Hello{
   def printMe( ) : Unit = {
      println("Hello, Scala!")
   }
}
```

### 方法调用

Scala可以使用多种方式调用方法：

- 调用方法的标准格式：

  ```scala
  functionName( 参数列表 )
  ```

- 使用实例对象调用方法：

  ```
  [instance.]functionName( 参数列表 )
  ```

示例：

```scala
object Test {
   def main(args: Array[String]) {
        println( "Returned Value : " + addInt(5,7) );
   }
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
```

## Scala闭包

闭包，是一个函数，其返回值依赖于声明在函数外部的一个或多个变量。可以理解为可以访问一个函数里面局部变量的另外一个函数

示例：

```scala
object Test {  
   def main(args: Array[String]) {  
      println( "muliplier(1) value = " +  multiplier(1) )  
      println( "muliplier(2) value = " +  multiplier(2) )  
   }  
   var factor = 3  
   val multiplier = (i:Int) => i * factor  
}  
```

> 函数mutiplier成为一个”闭包“，因为引用到函数外面定义的变量，定义这个函数的过程是将这个自由变量factor捕获而构成一个封闭的函数

## Scala数组

数组声明语法：

```
var z:Array[String] = new Array[String](3)

或

var z = new Array[String](3)
```

可以通过索引访问每个元素：

```scala
z(0) = "Runoob"; z(1) = "Baidu"; z(4/2) = "Google"
```

## Scala Collection

Scala集合分为可变和不可变集合：

- 可变集合，可以修改、添加、移除元素
- 不可变集合，永远不会改变

常用集合类型：

- List（列表）

  其元素以线性方式存储，集合中可以存放重复对象

- Set（集合）

  集合中的对象不按特定的方式排序，没哟重复对象

- Map（映射）

  Map是一种把键对象和值对象映射的集合，它的每一个元素都包含一个键对象和值对象

- 元组

  元组是不同类型的值得集合

- Option

  Option[T]表示有可能包含值得容器，也可能不包含

- Iterator（迭代器）

  迭代器不是一个容器，而是逐一访问容器内元素的方法

## Scala类和对象

类是对象的抽象，对象是类的具体实例

可以使用new关键字来创建类的对象

示例：

```scala
class Point(xc: Int, yc: Int) {
   var x: Int = xc
   var y: Int = yc

   def move(dx: Int, dy: Int) {
      x = x + dx
      y = y + dy
      println ("x 的坐标点: " + x);
      println ("y 的坐标点: " + y);
   }
}
```

> Scala中的类不声明为public，一个Scala源文件中可以有多个类。
>
> 以上实例的类定义了两个变量 **x** 和 **y** ，一个方法：**move**，方法没有返回值。
>
> Scala 的类定义可以有参数，称为类参数，如上面的 xc, yc，类参数在整个类中都可以访问。

使用new实例化类：

```scala
import java.io._

class Point(xc: Int, yc: Int) {
   var x: Int = xc
   var y: Int = yc

   def move(dx: Int, dy: Int) {
      x = x + dx
      y = y + dy
      println ("x 的坐标点: " + x);
      println ("y 的坐标点: " + y);
   }
}

object Test {
   def main(args: Array[String]) {
      val pt = new Point(10, 20);

      // 移到一个新的位置
      pt.move(10, 10);
   }
}
```

## Scala继承

Scala继承一个基类，需要注意几点：

- 重写一个非抽象方法必须使用override修饰符
- 在子类中重写超类的抽象方法时，不需要使用override关键字
- 只有主构造函数才可以往基类的构造函数里写参数

示例：

```scala
class Point(xc: Int, yc: Int) {
   var x: Int = xc
   var y: Int = yc

   def move(dx: Int, dy: Int) {
      x = x + dx
      y = y + dy
      println ("x 的坐标点: " + x);
      println ("y 的坐标点: " + y);
   }
}

class Location(override val xc: Int, override val yc: Int,
   val zc :Int) extends Point(xc, yc){
   var z: Int = zc

   def move(dx: Int, dy: Int, dz: Int) {
      x = x + dx
      y = y + dy
      z = z + dz
      println ("x 的坐标点 : " + x);
      println ("y 的坐标点 : " + y);
      println ("z 的坐标点 : " + z);
   }
}
```

> Scala 使用 extends 关键字来继承一个类。实例中 Location 类继承了 Point 类。Point 称为父类(基类)，Location 称为子类。
>
> **override val xc** 为重写了父类的字段

继承会继承父类的所有属性和方法，Scala值允许继承一个父类

示例：

```scala
import java.io._

class Point(val xc: Int, val yc: Int) {
   var x: Int = xc
   var y: Int = yc
   def move(dx: Int, dy: Int) {
      x = x + dx
      y = y + dy
      println ("x 的坐标点 : " + x);
      println ("y 的坐标点 : " + y);
   }
}

class Location(override val xc: Int, override val yc: Int,
   val zc :Int) extends Point(xc, yc){
   var z: Int = zc

   def move(dx: Int, dy: Int, dz: Int) {
      x = x + dx
      y = y + dy
      z = z + dz
      println ("x 的坐标点 : " + x);
      println ("y 的坐标点 : " + y);
      println ("z 的坐标点 : " + z);
   }
}

object Test {
   def main(args: Array[String]) {
      val loc = new Location(10, 20, 15);

      // 移到一个新的位置
      loc.move(10, 10, 5);
   }
}
```

Scala重写一个非抽象方法，必须写override修饰符

```scala
class Person {
  var name = ""
  override def toString = getClass.getName + "[name=" + name + "]"
}

class Employee extends Person {
  var salary = 0.0
  override def toString = super.toString + "[salary=" + salary + "]"
}

object Test extends App {
  val fred = new Employee
  fred.name = "Fred"
  fred.salary = 50000
  println(fred)
}
```

## Scala模式匹配

一个模式匹配包含了一系类备选项，每个都开始于关键字`case`。每个备选项都包含一个模式以及一到多个表达式，使用箭头符号`=>`隔开模式和表达式

示例：

```scala
object Test {
   def main(args: Array[String]) {
      println(matchTest(3))

   }
   def matchTest(x: Int): String = x match {
      case 1 => "one"
      case 2 => "two"
      case _ => "many"
   }
}
```

`match`对应java中的switch，但是卸载选择器表达式之后。即：**选择器 match {备选项}**

match表达式通过以代码编写的先后次序尝试每个模式来完成计算，只要发现有一个匹配的case，剩下的case不会继续匹配

## Scala文件I/O

Scala进行文件写操作，直接用的都是java中的I/O类(java.io.File)：

```scala
import java.io._

object Test {
   def main(args: Array[String]) {
      val writer = new PrintWriter(new File("test.txt" ))

      writer.write("菜鸟教程")
      writer.close()
   }
}
```

从屏幕读取用户输入:

```scala
import scala.io._
object Test {
   def main(args: Array[String]) {
      print("请输入菜鸟教程官网 : " )
      val line = StdIn.readLine()

      println("谢谢，你输入的是: " + line)
   }
}
```

从文件上读取内容：

```scala
import scala.io.Source

object Test {
   def main(args: Array[String]) {
      println("文件内容为:" )

      Source.fromFile("test.txt" ).foreach{ 
         print 
      }
   }
}
```

