# Tars入门

## 目录

- 服务开发
- 客户端调用服务
- 服务命名

## 服务开发

- 添加依赖

  ```xml
  <dependency>
      <groupId>qq-cloud-central</groupId>
      <artifactId>tars-server</artifactId>
      <version>1.0.1</version>
      <type>jar</type>
  </dependency>
  ```

- 添加插件

  ```xml
  <plugin>
      <groupId>qq-cloud-central</groupId>
      <artifactId>tars-maven-plugin</artifactId>
      <version>1.0.1</version>
      <configuration>
          <tars2JavaConfig>
              <tarsFiles>
                  <tarsFile>${basedir}/src/main/resources/hello.tars</tarsFile>
              </tarsFiles>
              <tarsFileCharset>UTF-8</tarsFileCharset>
              <servant>true</servant>
              <srcPath>${basedir}/src/main/java</srcPath>
              <charset>UTF-8</charset>
              <packagePrefixName>com.qq.tars.quickstart.server.</packagePrefixName>
          </tars2JavaConfig>
      </configuration>
  </plugin>
  ```

- 接口文件定义

  接口文件定义通过Tars接口描述语言来定义，在src/main/resources目录下建立hello.tars文件，内容如下：

  ```
  module TestApp 
  {
  	interface Hello
  	{
  	    string hello(int no, string name);
  	};
  };
  ```

- 接口文件编译

  提供插件编译生成java代码，在tars-maven-plugin添加生成java文件配置：

  ```xml
  <plugin>
  	<groupId>qq-cloud-central</groupId>
  	<artifactId>tars-maven-plugin</artifactId>
  	<version>1.0.1</version>
  	<configuration>
  		<tars2JavaConfig>
  			<!-- tars文件位置 -->
  			<tarsFiles>
  				<tarsFile>${basedir}/src/main/resources/hello.tars</tarsFile>
  			</tarsFiles>
  			<!-- 源文件编码 -->
  			<tarsFileCharset>UTF-8</tarsFileCharset>
  			<!-- 生成服务端代码 -->
  			<servant>true</servant>
  			<!-- 生成源代码编码 -->
  			<charset>UTF-8</charset>
  			<!-- 生成的源代码目录 -->
  			<srcPath>${basedir}/src/main/java</srcPath>
  			<!-- 生成源代码包前缀 -->
  			<packagePrefixName>com.qq.tars.quickstart.server.</packagePrefixName>
  		</tars2JavaConfig>
  	</configuration>
  </plugin>
  ```

- 执行maven命令

  ```
  mvn tars:tars2java
  ```

  生成代码如下：

  ```java
  @Servant
  public interface HelloServant {
  
  	public String hello(int no, String name);
  }
  ```

- 服务接口实现

  创建一个HelloServantImpl.java文件，实现HelloServant.java接口：

  ```java
  public class HelloServantImpl implements HelloServant {
  
  @Override
  public String hello(int no, String name) {
      return String.format("hello no=%s, name=%s, time=%s", no, name, System.currentTimeMillis());
  }
  ```

- 服务暴露配置

  在WEB-INF下创建一个servants.xml配置文件：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <servants>
  	<servant name="HelloObj">
  		<home-api>com.qq.tars.quickstart.server.testapp.HelloServant</home-api>
  		<home-class>com.qq.tars.quickstart.server.testapp.impl.HelloServantImpl</home-class>
  	</servant>
  </servants>
  ```

- 服务编译打包

  在工程目录下执行mvn package生成war包

## 客户端调用服务

- 添加依赖

  ```xml
  <dependency>
      <groupId>qq-cloud-central</groupId>
      <artifactId>tars-client</artifactId>
      <version>1.0.1</version>
      <type>jar</type>
  </dependency>
  ```

- 添加插件

  ```xml
  <plugin>
      <groupId>qq-cloud-central</groupId>
      <artifactId>tars-maven-plugin</artifactId>
      <version>1.0.1</version>
      <configuration>
          <tars2JavaConfig>
              <!-- tars文件位置 -->
              <tarsFiles>
                  <tarsFile>${basedir}/src/main/resources/hello.tars</tarsFile>
              </tarsFiles>
              <!-- 源文件编码 -->
              <tarsFileCharset>UTF-8</tarsFileCharset>
              <!-- 生成代码，PS：客户端调用，这里需要设置为false -->
              <servant>false</servant>
              <!-- 生成源代码编码 -->
              <charset>UTF-8</charset>
              <!-- 生成的源代码目录 -->
              <srcPath>${basedir}/src/main/java</srcPath>
              <!-- 生成源代码包前缀 -->
              <packagePrefixName>com.qq.tars.quickstart.client.</packagePrefixName>
          </tars2JavaConfig>
      </configuration>
  </plugin>
  ```

- 根据服务tars接口文件生成代码

  ```java
  @Servant
  public interface HelloPrx {
  
      public String hello(int no, String name);
  
      public String hello(int no, String name, @TarsContext java.util.Map<String, String> ctx);
  
      public void async_hello(@TarsCallback HelloPrxCallback callback, int no, String name);
  
      public void async_hello(@TarsCallback HelloPrxCallback callback, int no, String name, @TarsContext java.util.Map<String, String> ctx);
  }
  ```

- 同步调用

  ```java
  public static void main(String[] args) {
      CommunicatorConfig cfg = new CommunicatorConfig();
      //构建通信器
      Communicator communicator = CommunicatorFactory.getInstance().getCommunicator(cfg);
      //通过通信器，生成代理对象
      HelloPrx proxy = communicator.stringToProxy(HelloPrx.class, "TestApp.HelloServer.HelloObj");
      String ret = proxy.hello(1000, "HelloWorld");
      System.out.println(ret);
  }
  ```

- 异步调用

  ```java
  public static void main(String[] args) {
      CommunicatorConfig cfg = new CommunicatorConfig();
      //构建通信器
      Communicator communicator = CommunicatorFactory.getInstance().getCommunicator(cfg);
      //通过通信器，生成代理对象
      HelloPrx proxy = communicator.stringToProxy(HelloPrx.class, "TestApp.HelloServer.HelloObj");
      proxy.async_hello(new HelloPrxCallback() {
  
          @Override
          public void callback_expired() {
          }
  
          @Override
          public void callback_exception(Throwable ex) {
          }
  
          @Override
          public void callback_hello(String ret) {
              System.out.println(ret);
          }
      }, 1000, "HelloWorld");
  }
  ```

## 服务命名

Tars框架的服务，服务名称有三个部分：

- APP：应用名，表示一组服务的一个小集合，在Tars系统中，应用名必须唯一。例如：TestApp
- Server：服务名，提供服务的进程名称，Server名字根据业务服务功能命名，一般命名为：XXServer，例如：HelloServer
- Servant：服务者，提供具体服务的接口或实例，例如：HelloImp

一个Server可以包含多个Servant，系统会使用服务的App + Server + Servant 进行组合，来定义服务在系统中的路由名称，称为路由Obj，其名称在系统中必须是唯一的。例如：TestApp.HelloServer.HelloObj