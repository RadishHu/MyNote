# Spring基础

Spring两大核心：DI(Dependency Injection,依赖注入)，AOP(Aspect-Oriented Programming，面向切面编程)

## DI-依赖注入

装配(wiring)，创建应用组件之间协作的行为

Spring装配bean的方式：

- 基于XML文件装配

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="knight" class="sia.knights.BraveKnight">
      <constructor-arg ref="quest" />
    </bean>
    <bean id="quest" class="sia.knights.SlayDragonQuest">
      <constructor-arg value="#{T(System).out}" />
    </bean>
  </beans>
  ```

- 基于Java装配

  ```java
  package sia.knights.config;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  import sia.knights.BraveKnight;
  import sia.knights.Knight;
  import sia.knights.Quest;
  import sia.knights.SlayDragonQuest;
  
  @Configuration
  public class KnightConfig {
  
    @Bean
    public Knight knight() {
      return new BraveKnight(quest());
    }
    
    @Bean
    public Quest quest() {
      return new SlayDragonQuest(System.out);
    }
  }
  ```

## Spring Container

Spring Container，容器，容器负责创建应用程序中的bean，并通过DI来装配它们，并管理它们的整个生命周期。

Spring容器分为两种不同的类型：

- Bean Factory，由org.springframework.beans.factory.beanFactory接口定义，它是最简单的容器，提供基本的DI支持。
- Application Context，应用上下文，ioioioioooiopoppoop由org.springframework.context.ApplicationContext接口定义，它是基于BeanFactory构建，提供应用框架级别的服务。

Application Context，装载在bean的定义并把它们组装起来，它负责对象的创建和组装，Spring自带了多种应用上下文的实现，它们之间的主要区别在于如何加载配置。

Spring常用的Application Context：

- AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文
- AnnotationConfigWebApplicationContext：从一个或多个基于Java的配置类中加载SpringWeb应用上下文
- ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源
- FileSystemXmlApplicationContext：从文件系统下的一个或多个XML配置文件中加载上下文定义
- XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件中加载上下文定义

Spring通过Application Context装载bean的定义并把它们组装在一起。

```java
package sia.knights;

import org.springframework.context.support.
                   ClassPathXmlApplicationContext;

public class KnightMain {

  public static void main(String[] args) throws Exception {
    ClassPathXmlApplicationContext context = 
        new ClassPathXmlApplicationContext(
            "META-INF/spring/knight.xml");
    Knight knight = context.getBean(Knight.class);
    knight.embarkOnQuest();
    context.close();
  }

}
```

## Spring模块

Spring4.0中包括20个不同的模块，每个模块有三个jar文件：二进制类库、源码的jar文件、JavaDoc的jar文件。

这些模块依据所属的功能可以划分为6类不同的功能：

![](.\images\Spring功能模块.png)

- Spring核心容器

  容器是Spring框架的核心部分，它管理Spring应用中bean的创建、配置和管理。

- AOP模块

  Aop模块对面向切面编程提供了丰富的支持

- 数据访问与集成模块

  使用JDBC编写代码通常会导致大量的样板式代码。Spring的JDBC和DAO模块抽象了这些样板式代码。

  本模块会使用Spring AOP模块为Spring应用中的对象提供事务管理服务

- Web与远程调用模块

  MVC(Model-View-Controller)模式是一种常用的构建Web应用的方法，Spring的Web和远程调用模块自带了一个MVC框架

- Instrumentation

  该模块提供了为JVM添加代理(agent)的功能

- 测试

POJO，Plain Old Java Object

DAO，Data Access Object