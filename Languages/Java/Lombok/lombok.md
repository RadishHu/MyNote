# Lombok

lombok可以在项目编译的时候生成一些代码，它可以通过注解来标识生成`getter`  、`setter`等代码

## 引入依赖

- Maven依赖

  ```xml
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.16.20</version>
  </dependency>
  ```

- IDEA安装插件

  settings --> plugins --> lombok

## 基本用法

- Getter Setter

  ```java
  package com.test.lombok;
  
  import lombok.AccessLevel;
  import lombok.Getter;
  import lombok.Setter;
  
  public class GetterSetterExample {
      @Getter
      @Setter
      private int age = 10;
      
      @Getter
      @Setter
      private boolean active;
  
      @Getter
      @Setter
      private Boolean none;
      
      @Setter(AccessLevel.PROTECTED) 
      private String name;
  }
  ```

  - @Getter声明创建getter方法
  - @Setter声明创建setter方法
  - @Setter(AccessLevel.PROTECTED)指定权限为私有
  - boolean的set前缀都为set，但是getter不同，小写boolean，即基本类型，编译后方法名前缀是is；大写Boolean，即包装类型，编译后方法名前缀是get

- ToString

  ```java
  package com.test.lombok;
  
  import lombok.Setter;
  import lombok.ToString;
  
  @Setter
  @ToString(exclude="id")
  public class ToStringExample {
      private static final int STATIC_VAR = 10;
      private String name;
      private Shape shape = new Square(5, 10);
      private String[] tags;
      private int id;
  }
  ```

- Equals、HashCode

  ```java
  package com.test.lombok;
  
  import lombok.EqualsAndHashCode;
  
  @EqualsAndHashCode(exclude={"id", "shape"})
  public class EqualsAndHashCodeExample {
      private transient int transientVar = 10;
      private String name;
      private double score;
      private ToStringExample.Shape shape = new Square(5, 10);
      private String[] tags;
      private int id;
  }
  ```

- 构造函数

  - @NoArgsConstructor,生成一个无参构造器

    ```java
    @NoArgsConstructor
    public static class NoArgsExample {
    }
    ```

  - @RequiredArgsConstructor

    生成只有某几个字段的构造器，对要构造器中要包含的字段标注`@NonNull`,标注的字段不应为null，初始化的时候回检查是否为空，为空抛出`NullPointException`

    ```java
    @RequiredArgsConstructor
    public class RequiredArgsExample {
        @NonNull private String field;
        private Date date;
        private Integer integer;
        private int i;
        private boolean b;
        private Boolean aBoolean;
    }
    ```

  - @AllArgsConstructor

    初始化所有字段

    ```java
    @AllArgsConstructor(access = AccessLevel.PROTECTED)
    public class ConstructorExample<T> {
        private int x, y;
        @NonNull
        private T description;
    }
    ```

- @Data

  @Data是一个集合，包含Getter、Setter、RequiredArgsConstructor、ToString、EqualsAndHashCode

- @Value

  @Value是一个集合体，包含Getter、AllArgsConstructor、ToString、EqualsAndHashCode

- @Builder

  声明式简化对象创建流程



参考：https://www.cnblogs.com/woshimrf/p/lombok-usage.html

