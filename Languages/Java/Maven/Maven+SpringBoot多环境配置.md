# Maven+SpringBoot多环境配置

- 配置application.properties内容：

  ```properties
  spring.profiles.active=@profileActive@
  ```

- 创建不同的环境下的配置文件：

  - application-dev.properties：开发环境
  - application-test.properties：测试环境
  - application-prod.properties：生产环境 

- 在pom.xml文件中配置profiles：

  ```xml
  <profiles>
  	<profile>
  		<id>dev</id>
  		<activation>
  			<activeByDefault>true</activeByDefault>
  		</activation>
  		<properties>
  			<profileActive>dev</profileActive>
  		</properties>
  	</profile>
  	<profile>
  		<id>test</id>
  		<properties>
  			<profileActive>test</profileActive>
  		</properties>
  	</profile>
  	<profile>
  		<id>prod</id>
  		<properties>
  			<profileActive>prod</profileActive>
  		</properties>
  	</profile>
  </profiles>
  ```

- 使用maven命令打包：

  ```
  mvn clean package -P prod -U
  ```

- IDEA中运行项目：

  在idea中直接运行项目会报错：

  ```
  ava.lang.IllegalArgumentException: Could not resolve placeholder
  ```

  需要在启动参数添加：

  ```
  --spring.profiles.active=dev
  ```

## 扩展内容

> * [SpringBoot多环境配置](/Languages/Java/SpringBoot/SpringBoot多环境配置.md)
> * [Maven-pom文件](/Languages/Java/Maven/pom文件.md)

