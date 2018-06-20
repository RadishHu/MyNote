# Spring Boot配置日志

Spring Boot默认使用Logback来记录日志，并用INFO级别输出到控制台

## 用其它日志替换Logback

使用Log4j或者Log4j2替换Logback

- 排除根起步依赖传递引入的默认日志起步依赖：

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <exclusions>
          <exclusion>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-logging</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```

- 添加Log4j

  ```xml
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-log4j</artifactId>
  </dependency>
  ```

  如果要用Log4j2，可以把spring-boot-starter-log4j改为spring-boot-starter-log4j2

## 修改日志属性的配置

- 设置日志级别

  设置本日志级别为WARN，但Spring Security的日志级别为DEBUG，可以修改application.yml文件为：

  ```yml
  logging:
  	level:
  	root: WARN
  	org:
  		springframework:
  			security: DEBUG
  ```

- 设置日志输出文件：

  yml文件

  ```yml
  logging:
  	path: /var/logs
  	file: BookWorm.log
  	level:
  		root: WARN
  		org:
  			springframework:
  				security: DEBUG
  ```

  properties文件

  ```properties
  logging.path=/var/logs/
  logging.file=BookWorm.log
  logging.level.root=WARN
  logging.level.root.org.springframework.security=DEBUG
  ```

- 设置日志配置文件

  ```
  logging：
  	config：
  		classpath:logging-config.xml
  ```

