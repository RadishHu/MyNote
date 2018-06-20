# Spring Boot自动配置

## 覆盖自动配置

- 覆盖自动配置

  要覆盖Spring Boot自动配置，只要编写一个显式的配置就可以了。Spring Boot会发现显式配置，并降低自动配置的优先级。

- 覆盖自动配置的原理

  @ConditionalMissingBean注解是覆盖自动配置的关键。

  以Spring Boot的DataSourceAutoConfiguration中定义的Jdbc Template Bean为例：

  ```java
  @Bean
  @ConditionalOnMissingBean(JdbcOperations.class)
  public JdbcTemplate jdbcTemplate() {
  	return new JdbcTemplate(this.dataSource);
  }
  ```

  JdbcTemplate()方法上添加了@Bean注解，在需要时可以配置出一个JdbcTemplateBean。它上面还添加了@ConditionalMissingBean注解，要求不存在JdbcOperations类型(JdbcTemplate实现了该接口)Bean时才生效，如果当前已经有一个JdbcOperations Bean了，条件不满足，不会执行jdbcTemplate()方法。

## 自动配置微调

为了微调一些细节，比如改端口号和日志级别，如果为了微调就启用自动配置是没有必要的，Spring Boot自动配置的Bean提供了300多个用于微调的属性。

Spring Boot应用程序有多重设置微调属性的途径：

- 命令行参数
- java:comp/env里的JNDI属性
- JVM系统属性
- 操作系统环境变量
- 随机生成的带random.*前缀的属性
- 应用程序以外的application.properties或者application.yml文件
- 打包在应用程序内的application.properties或者application.yml文件
- 通过@PropertySource标注的属性源
- 默认属性

这个列表按优先级排序，如果同时有application.properties和application.yml文件，那么yml文件里的属性会覆盖properties文件中属性

微调属性：

- 修改嵌入式服务器的端口：

  server.port=8000

- 开启嵌入式服务器的HTTPS服务：

  - 用JDK的keytool工具来创建一个秘钥存储：

    $ keytool -keystore mykeys.jks -genkey -alias tomcat -keyalg RSA

  - 在application.properties或application.yml文件中设置属性：

    ```yml
    server:
    	port: 8843
    	ssl:
    		key-store: file:///path/tomykeys.jks
    		key-store-password: letmain
    		key-password: lemain
    ```

    开发中https服务大多会选这个端口，server.ssl.key-store属性指向秘钥存储文件的存放路径

