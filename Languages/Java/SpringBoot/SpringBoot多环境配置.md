# SpringBoot多环境配置

## properties配置

多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应环境标识：

- application-dev.properties：开发环境
- application-test.properties：测试环境
- application-prod.properties：生产环境  

具体使用哪个配置文件，需要在application.properties文件中通过spring.profiles.active属性来设置，其值对应`{profile}`值：

```properties
spring.profiles.active=dev
```

## 使用不同配置

>  执行java -jar xxx.jar，使用默认开发环境(dev)
>
> 执行java -jar xxx.jar --spring.profiles.active=test，使用测试环境(test)

