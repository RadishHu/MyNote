# 装配Bean

Spring装配Bean的三种机制：

- 在XML中进行显式配置
- 在Java中进行显式配置
- 隐式的bean发现机制和自动装配

## Spring自动化装配Bean

- 开启组件扫描(component scanning)：Spring自动发现应用上下文中所创建的bean

  - 使用Java进行装配，使用@ComponentScan注解开启组件扫描

    ```java
    package soundsystem;
    
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    @Configuration
    @ComponentScan
    public class CDPlayerConfig {
    }
    ```

  - 使用XML进行装配，用Spring context命名空间的\<context:component-scan>元素开启组件扫描

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xmlns:c="http://www.springframework.org/schema/c"
      xmlns:p="http://www.springframework.org/schema/p"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
      <context:component-scan base-package="soundsystem" />
    
    </beans>
    ```

- 自动装配(autowiring)：Spring自动满足bean之间的依赖

## Java装配Bean