# Spring Boot应用程序部署

- 修改pom.xml中的打包方式

  ```xml
  <packaging>war</packaging>
  ```

- 使用Spring Boot的 SpringBootServletInitializer

  先前打成的WAR文件中没有启用Spring MVC DispatcherServlet的web.xml文件和Servlet初始类。

  SpringBootServletInitializer 是一个支持Spring Boot的Spring WebApplicationInitializer 实现。除了配置Spring的 Dispatcher-Servlet ，SpringBootServletInitializer 还会在Spring应用程序上下文里查找 Filter 、
  Servlet 或 ServletContextInitializer 类型的Bean，把它们绑定到Servlet容器里

  使用Spring Boot ServletInitializer,只需要创建一个子类，覆盖configure()方法来指定Spring配置类：

  ```java
  package readinglist;
  import org.springframework.boot.builder.SpringApplicationBuilder;
  import org.springframework.boot.context.web.SpringBootServletInitializer;
  
  public class ReadingListServletInitializer extends SpringBootServletInitializer {
      @Override
      protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
      	return builder.sources(Application.class); //指定配置类
      }
  }
  ```

