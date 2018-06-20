# Spring Boot配置数据源

## 配置MySQL数据库

修改application.yml文件：

```
spring:
  datasource:
    url: jdbc:mysql://localhost/database
    username: dbuser
    password: dbpass
    driver-class-name: com.mysql.jdbc.Driver
```

