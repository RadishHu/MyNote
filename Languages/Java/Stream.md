# Stream

## Stream简介

Java 8中引入流(Stram)的概念，所有继承自Collection的接口都可以转换为Stream

示例：

```java
//List中包含一系列Person，Person中有姓名name和年龄age字段
long count = 0;
for (Person p : persons) {
    if (p.getAge() > 20) {
        count ++;
    }
}

//使用Stream实现相同的功能
long count = person.stream()
    	.filter(person -> person.getAge() > 20)
    	.count();
```

## Stream常用操作

Stream的方法分为两类：惰性求值，及早求值

判断一个造作值惰性求值还是及早求职：看返回值，如果返回值是Stream，那么就是惰性求值

- collection(toList())

  collection(toList())方法由Stream里的值生成一个列表，可以理解为Stream向Collection的转换

  ```java
  List<String> collected = Stream.of("a","b","c").collect(toList());
  ```

- map

  map()方法接受一个Function的匿名函数类，它使用传入的函数将一个流中的值转换为另一个新的流

  ```java
  List<String> collected = Stream.of("a","b","hello")
      	.map(string -> string.toUpperCase())
      	.collect(toList());
  ```

- filter

  filter()方法接受一个Predicate的匿名函数类，判断对象是否符合条件，符合条件的才保留下来

  ```java
  List<String> beginningWithNumbers = 
      Stream.of("a","labc","abcl")
      .filter(value -> isDigit(value.charAt(0)))
      .collect(toList);
  ```

- flatMap

  flatMap()方法可以将多个Stream合并为一个Stream

  ```java
  List<Integer> together = Stream.of(asList(1,2),asList(3,4))
      .flatMap(numbers -> numbers.stream())
      .collect(toList());
  ```

- max和min

  求最大值和最小值

  ```java
  List<Integer> list = Lists.newArrayList(3,5,2,9,1);
  int maxInt = list.stream()
      .max(Integer::compareTo)
      .get();
  int minInt = list.stream()
      .min(Integer::compareTo)
      .get();
  ```

  max()和min()方法返回的是一Optional对象，Optional对象封装了实际值

- reduce

  reduce()可以从一组值中生成一个值

  ```java
  int result = Stream.of(1,2,3,4)
      .reduce(0,(acc,element) -> acc + element);
  ```

  reduce的第一参数，是一个初始值，即0 + 1 + 2 + 3 + 4 = 10

## 数据并行化操作

数据并行化是指将数据分成块，为每块数据分配单独处理单元。

并行化操作流，调用Stream对象的parallel()方法，让它拥有并行操作的能力。如果从集合类创建一个流，调用parallelStream()就可以获得一个拥有并行能力的流。

```java
int sumSize = Stream.of("Apple","Banana","Orange","Pear")
    .parallel()
    .map(s -> s.length())
    .reduce(Integer::sum)
    .get();
```

加上parallel()方法，对数据进行分块，然后对每块数据开辟线程进行运算，这些操作会花费额外的时间。并行化操作只有在数据规模较大或数据处理时间较长的时候才能提高效率。

