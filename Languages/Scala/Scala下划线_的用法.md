# Scala下划线“_"的用法

- 初始化的时候

  ```scala
  object Sample {
     var name:String=_
     def main (args: Array[String]){
        name="hello world"
       println(name)
     }
  }
  ```

  > 将name赋值为null，这里_与null作用一样，相当于var name: String = null

- 引入的时候

  ```scala
  import math._
  object Sample {
     def main (args: Array[String]){
      println(BigInt(123))
     }
  }
  ```

  > 这里math._相当于Java中的math.*，即引用包中所有内容

- 集合中

  ```scala
  object Sample {
     def main (args: Array[String]){
      val newArry = (1 to 10).map(_*2)
     println(newArry)
     }
  }
  ```

  > 这里下划线代表集合中的某一个元素，在foreach中也可以使用

- 模式匹配

  ```scala
  object Sample {
     def main (args: Array[String]){
       val value="a"
    val result=  value match{
         case "a" => 1
         case "b" => 2
         case _ =>"result"
       }
       println(result)
     }
  }
  ```

  > 这里的下划线相当于others的意思

  被Some包起来，说明Some里面有值的，而不是None

  ```scala
  object Sample {
    def main (args: Array[String]){
      val value=Some("a")
      val result=  value match{
        case Some(_) => 1
        case _ =>"result"
      }
      println(result)
    }
  }
  ```

  表示队列

  ```scala
  object Sample {
    def main (args: Array[String]){
      val value=1 to 5
      val result=  value match{
        case Seq(_,_*) => 1
        case _ =>"result"
      }
      println(result)
    }
  }
  ```

- 函数中使用

  ```scala
  object Sample {
     def main (args: Array[String]){
      val set=setFunction(3.0,_:Double)
       println(set(7.1))
     }
    def setFunction(parm1:Double,parm2:Double): Double = parm1+parm2
  }
  ```

  > 这是Scala中“偏函数”用法

- 元组Tuple

  ```scala
  object Sample {
     def main (args: Array[String])={
       val value=(1,2)
       print(value._1)
     }
  }
  ```

- 传参

  ```scala
  object Sample {
     def main (args: Array[String])={
      val result=sum(1 to 5:_*)
       println(result)
     }
    def sum(parms:Int*)={
      var result=0
      for(parm <- parms)result+=parm
      result
    }
  }
  ```

  > 当函数接收的参数不定长的时候，假如你想输入一个队列，可以在一个队列后加入“:_*”，因此，这里的“1 to 5”也可以改写为：“Seq(1,2,3,4,5)”

