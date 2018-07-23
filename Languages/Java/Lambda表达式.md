# Lambda表达式

参考：

> * [Lambda Expressions](https://blog.csdn.net/lsgqjh/article/details/54582945)

## 目录

> * [简介](#chapter1)
> * [函数式接口](#chapter2)
> * [Lambda表达式语法](#chapter3)
> * [Lambda表达式使用示例](#chapter4)
> * [使用Lambds和Streams](#chapter5)

## 简介 <a id="chapter1"></a>

Lambda表达式 用一个表达式实现了 单个接口方法(函数式接口)

Lambda表达式长用作匿名内部类的替代

## 函数式接口 <a id="chapter2"></a>

Lambdas只能在一个仅包含一个抽象方法的函数式接口上操作

函数式接口指具有单个抽象方法的接口：

```java
interface Foo1 {
    void bar();
}

interface Foo2 {
    int bar(boolean baz);
}

interface Foo3 {
    String bar(Object baz, int mink);
}

interface Foo4 {
    default String bar() { // default so not counted
        return "baz";
    }
    void quux();
}
```

在盛明明函数式接口时，可以添加一个`@FunctionalInterface`注解，如果这个注解被用于非函数式接口，会产生`compiler error`：

```java
@FunctionalInterface
interface Foo5 {
    void bar();
}

@FunctionalInterface
interface BlankFoo1 extends Foo3 { // inherits abstract method from Foo3
}

@FunctionalInterface
interface Foo6 {
    void bar();
    boolean equals(Object obj); // overrides one of Object's method so not counted
}
```

非函数式接口：

```java
//多个抽象方法
interface BadFoo {
    void bar();
    void quux(); // <-- Second method prevents lambda: which one should be considered as lambda?
}

//没有抽象方法
interface BlankFoo2 { }
```

## Lambda表达式语法 <a id="chapter3"></a>

Lambda表达式的基本结构：

```
FunctionalInterface fi = () -> System.out.println("hello");
```

> * ()：Method Signature
> * ->：Lambda Operator
> * System.out.println("hello")：Method Implementation

`fi`持有一个实现了`FunctionalInterface`接口的匿名类的实例，等价于：

```java
FunctionalInterface fi = new FunctionalInterface() {
    @Override
    public void theOneMethod() {
        System.out.println("Hello");
    }
};
```

示例：

```java
//不需要参数，返回值为5
() -> 5

//接受一个参数(数字类型)，返回其2倍的值
x -> 2 * x

//接受2个参数，返回它们的差值
(x,y) -> x - y

//接受2个int型的整数，返回它们的和
(int x,int y) -> x + y

//接受一个string对象，在控制台打印，不返回任何值
(String s) -> System.out.println(s)
```

## Lambda表达式使用示例 <a id="chapter4"></a>

- 遍历List集合

  ```java
  String[] atp = {"Rafael Nadal", "Novak Djokovic",  
         "Stanislas Wawrinka",  
         "David Ferrer","Roger Federer",  
         "Andy Murray","Tomas Berdych",  
         "Juan Martin Del Potro"};  
  List<String> players =  Arrays.asList(atp);  
    
  // 以前的循环方式  
  for (String player : players) {  
       System.out.print(player + "; ");  
  }  
    
  // 使用 lambda 表达式以及函数操作(functional operation)  
  players.forEach((player) -> System.out.print(player + "; "));  
     
  // 在 Java 8 中使用双冒号操作符(double colon operator)  
  players.forEach(System.out::println); 
  ```

- 实现Runnable接口：

  ```java
  //使用匿名内部类
  new Thread(new Runnable() {
       @Override
      public void run() {
          System.out.println("Hello World!");
      }
  }).start();
  
  //使用lambda expression
  new Thread(() -> System.out.println("Hello World!")).start();
  
  //使用匿名内部类
  Runnable race1 = new Runnable() {
      @Override
      public void run() {
          System.out.println("Hello World!");
      }
  }
  
  //使用lambda expression
  Runnable race2 = () -> System.out.println("Hello World!");
  
  race1.run();
  race2.run();
  ```

- Lambda排序集合

  ```java
  String[] players = {"Rafael Nadal", "Novak Djokovic",   
      "Stanislas Wawrinka", "David Ferrer",  
      "Roger Federer", "Andy Murray",  
      "Tomas Berdych", "Juan Martin Del Potro",  
      "Richard Gasquet", "John Isner"};
  //使用匿名内部类根据name排序players
  Arrays.sort(players,new Comarator<String>() {
     @Override
      public int compare(String s1,String s2) {
          return (s1.compareTo(s2));
      }
  });
  
  //使用lambda expression排序players
  Comparator<String> sortByName = (String s1,String s2) -> (s1.compareTo(s2));
  Arrays.sort(players,sortByName);
  
  //也可以使用如下：
  Arrays.sort(players,(String s1,String s2) -> (s1.compareTo(s2)));
  ```

## 使用Lambds和Streams <a id="chapter5"></a>

Stream是对集合的包装，它是懒计算。

使用示例：

- 定义Person类：

  ```java
  public class Person {  
    
      private String firstName, lastName, job, gender;  
      private int salary, age;  
  
      public Person(String firstName, String lastName, String job,  
                      String gender, int age, int salary)       {  
                this.firstName = firstName;  
                this.lastName = lastName;  
                this.gender = gender;  
                this.age = age;  
                this.job = job;  
                this.salary = salary;  
      }  
  // Getter and Setter   
  // . . . . .  
  }
  ```

- 创建list，存放Person

  ```java
  List<Person> javaProgrammers = new ArrayList<Person>() {  
    {  
      add(new Person("Elsdon", "Jaycob", "Java programmer", "male", 43, 2000));  
      add(new Person("Tamsen", "Brittany", "Java programmer", "female", 23, 1500));  
      add(new Person("Floyd", "Donny", "Java programmer", "male", 33, 1800));  
      add(new Person("Sindy", "Jonie", "Java programmer", "female", 32, 1600));  
      add(new Person("Vere", "Hervey", "Java programmer", "male", 22, 1200));  
      add(new Person("Maude", "Jaimie", "Java programmer", "female", 27, 1900));  
      add(new Person("Shawn", "Randall", "Java programmer", "male", 30, 2300));  
      add(new Person("Jayden", "Corrina", "Java programmer", "female", 35, 1700));  
      add(new Person("Palmer", "Dene", "Java programmer", "male", 33, 2000));  
      add(new Person("Addison", "Pam", "Java programmer", "female", 34, 1300));  
    }  
  };  
    
  List<Person> phpProgrammers = new ArrayList<Person>() {  
    {  
      add(new Person("Jarrod", "Pace", "PHP programmer", "male", 34, 1550));  
      add(new Person("Clarette", "Cicely", "PHP programmer", "female", 23, 1200));  
      add(new Person("Victor", "Channing", "PHP programmer", "male", 32, 1600));  
      add(new Person("Tori", "Sheryl", "PHP programmer", "female", 21, 1000));  
      add(new Person("Osborne", "Shad", "PHP programmer", "male", 32, 1100));  
      add(new Person("Rosalind", "Layla", "PHP programmer", "female", 25, 1300));  
      add(new Person("Fraser", "Hewie", "PHP programmer", "male", 36, 1100));  
      add(new Person("Quinn", "Tamara", "PHP programmer", "female", 21, 1000));  
      add(new Person("Alvin", "Lance", "PHP programmer", "male", 38, 1600));  
      add(new Person("Evonne", "Shari", "PHP programmer", "female", 40, 1800));  
    }  
  };  
  ```

- 使用forEach，打印信息：

  ```java
  //打印所有程序员姓名
  javaProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
  phpProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
  ```

- 使用forEach，增加程序员工资：

  ```java
  Consumer<Person> giveRaise = e -> e.setSalary(e.getSalary() / 100 * 5 + e.getSalary());
  javaProgrammers.forEach(giveRaise);
  phpProgrammers.forEach(giveRaise);
  ```

- 使用过滤器filter()，显示月薪超过1400美元的PHP程序员：

  ```java
  phpProgrammers.stream()
  	.filter((p) -> (p.getSalary() > 1400))
  	.forEach((p) -> System.out.printf("%s %s;",p.getFirstName(),p.getLastName));
  ```

- 先定义过滤器，然后使用它执行操作：

  ```java
  //定义filters
  Predicate<Person> ageFilter = (p) -> (p.getAge() > 25);
  Predicate<Person> salaryFilter = (p) -> (p.getSalary() > 1400);
  Predicate<Person> genderFilter = (p) -> ("female".equals(p.getGender()));
  
  //使用定义好的过滤器
  phpProgrammers.stream()  
            .filter(ageFilter)  
            .filter(salaryFilter)  
            .filter(genderFilter)  
            .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
  ```

- 使用limit()方法，限制结果集的个数：

  ```java
  javaProgrammers.stream()
      .limit(3)
      .forEach((p) -> System.out.printf("%s %s;",p.getFirstName(),p.getLastName));
  ```

- 使用sorted()方法进行排序：

  ```java
  //根据name进行排序
  List<Person> sortedJavaProgrammers = javaProgrammers
  	.stream()
      .sorted((p,p2) -> (p.getFirstName().compareTo(p2.getFistName())))
      .limit(5)
      .collect(toList());
  
  //根据salary进行排序
  sortedJavaProgrammers = javaProgrammers
  	.stream()
      .sorted((p,p2) -> (p.getSalary() - p2.getSalary()))
      .collect(toList());
  ```

- min(),max()

  ```java
  //获取最低工资
  Person pers = javaProgrammers
  	.stream()
      .min((p1,p2) -> (p1,getSalary() - p2.getSalary()))
      .get();
  //获取最高工资
  Person pers = javaProgrammers
  	.stream()
      .max((p1,p2) -> (p1.getSalary() - p2.getSalary()))
      .get();
  ```