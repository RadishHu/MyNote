# Tars开发

## 目录

> * [服务开发](#chapter-1)
> * [客户端调用服务](#chapter-2)
> * [服务命名](#chapter-3)

## 服务开发 <a id="chapter-1"></a>

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

## 客户端调用服务 <a id="chapter-2"></a>

- 添加依赖

  ```xml
  
  ```

- 添加插件

  ```xml
  
  ```

- 根据服务tars接口文件生成代码

  ```java
  
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





