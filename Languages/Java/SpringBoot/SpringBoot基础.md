# Spring Boot基础知识

## Spring Boot项目目录结构

- pom.xml
- Application.java：一个带有main()方法的类，用于引导启动应用程序
- ApplicationTests.java：一个空的JUnit测试类，它加载一个使用Spring Boot自动配置功能的Spring应用程序上下文
- application.properties：一个空的properties文件，可以根据需要添加配置属性
- static目录：放置Web应用程序的静态内容(JavaScript、样式表、图片等)

## 常用注解

- @SpringBootApplication

  开启Spring的组件扫描和Spring Boot的自动配置功能

  @SpringBootApplication将三个注解组合在了一起：

  - Spring的@Configuration：标明该类使用Spring基于Java的配置
  - Spring的@ComponentScan：启用组件扫描，这样写的Web控制器类和其它组件才能被自动发现并注册为Spring应用程序上下文里的Bean
  - Spring Boot的@EnableAutoConfiguration：开启Spring Boot自动配置

- @Entity

  标注该类是一个JPA实体类

- @Id

  标注实体类的唯一标识

- @GeneratedValue

  标注标识符的生成策略，常于@Id一起使用，用于自动生成唯一标识的字段值

- @PostMapping

- @RequestBody

## application.properties文件

- 修改Tomcat监听的端口

  ```properties
  server.port=8000
  ```




