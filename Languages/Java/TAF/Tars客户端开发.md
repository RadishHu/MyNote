# Tars客户端开发

## 目录

> * [客户端开发](#chapter1)

## 客户端开发 <a id="chapter1"></a>

### 配置依赖

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

### 根据服务接口文件生成代码

```java
@Servant
public interface HelloPrx {

    public String hello(int no, String name);

    public String hello(int no, String name, @TarsContext java.util.Map<String, String> ctx);

    public void async_hello(@TarsCallback HelloPrxCallback callback, int no, String name);

    public void async_hello(@TarsCallback HelloPrxCallback callback, int no, String name, @TarsContext java.util.Map<String, String> ctx);
}
```

### 通信器

完成服务端后，客户端对服务端完成收发包的操作是通过通信器Communicator来实现的

出事话通信器：

```java
CommunicatorConfig cfg = CommunicatorConfig.load("config.conf");
//构建通信器
Communicator communicator = CommunicatorFactory.getInstance().getCommunicator(cfg);
```

说明：

> * 通信器不load配置文件的话，所有参数都采用默认值
> * 通信器可以通过属性来完成初始化
> * 如果需要通过名字来获取客户端调用代理，则必须设置locator参数

通信器属性说明：

> * locator: registry服务的地址，必须是有ip port的，如果不需要registry来定位服务，则不需要配置；
> * connect-timeout：网络连接超时时间，毫秒，没有配置缺省为3000
> * connections；连接数，默认为4；
> * sync-invoke-timeout：调用最大超时时间（同步），毫秒，没有配置缺省为3000
> * async-invoke-timeout：调用最大超时时间（异步），毫秒，没有配置缺省为5000
> * refresh-endpoint-interval：定时去registry刷新配置的时间间隔，毫秒，没有配置缺省为1分钟
> * stat：模块间调用服务的地址，如果没有配置，则上报的数据直接丢弃；
> * property：属性上报地址，如果没有配置，则上报的数据直接丢弃；
> * report-interval：上报给stat/property的时间间隔，默认为60000毫秒；
> * modulename：模块名称，默认为可执行程序名称；

通信器配置文件格式：

```
<tars>
  <application>
	#set调用
	enableset                      = N
	setdivision                    = NULL 
    #proxy需要的配置
    <client>
        #地址
        locator                     = tars.tarsregistry.QueryObj@tcp -h 127.0.0.1 -p 17890
        #同步最大超时时间(毫秒)
        connect-timeout             = 3000
        #网络连接数
        connections                 = 4
        #同步最大超时时间(毫秒)
        sync-invoke-timeout         = 3000
        #异步最大超时时间(毫秒)
        async-invoke-timeout        = 5000
        #刷新端口时间间隔(毫秒)
        refresh-endpoint-interval   = 60000
        #模块间调用
        stat                        = tars.tarsstat.StatObj
        #属性上报地址
        property                    = tars.tarsproperty.PropertyObj
        #report time interval
        report-interval             = 60000
        #模块名称
        modulename                  = TestApp.HelloServer
    </client>
  </application>
</tars>
```

### 超时控制

超时控制是对客户端proxy而言的，在上面的通信器配置文件中进行配置：

```
#同步最大超时时间(毫秒)
sync-invoke-timeout          = 3000
#异步最大超时时间(毫秒)
async-invoke-timeout         = 5000
```

上面的超时时间是对通信器生成的所哟proxy都有效，如果需要单独设置超时 时间，可以使用如下设置：

```java
//设置该代理单独初始化配置
public <T> T stringToProxy(Class<T> clazz, ServantProxyConfig servantProxyConfig)；
//ServantProxyConfig与CommunicatorConfig类似
```

### 调用

#### 寻址方式

Tars服务的寻址方式分为两种：服务名在主控注册和不在主控注册，主控是指专用于注册服务节点信息的名字服务(路由服务)

- 没有在主控注册的服务，在服务的obj后面指定要访问的ip地址。客户端在调用的时候需要指定HelloObj对象的具体地址：

  ```
  TestApp.HelloServer.HelloObj@tcp -h 127.0.0.1 -p 9985
  ```

  > TestApp.HelloServer.HelloObj：对象名称
  >
  > tcp：tcp协议
  >
  > -h：指定主机地址
  >
  > -p：端口地址

  如果HelloServer在两台服务器上运行，HelloPrx初始化的方式：

  ```java
  HelloPrx prx = c.stringToProxy("TestApp.HelloServer.HelloObj@tcp -h 127.0.0.1 -p 9985:tcp -h 192.168.1.1 -p 9983");
  ```

- 在主控中注册的服务，服务的寻址是基于服务名进行的，客户端在请求服务端的时候不需要指定HelloServer的具体地址，但是需要在生成通信器或初始化通信器的时候指定registry(主控中心)的地址

  ```java
  HelloPrx prx = c.stringToProxy<HelloPrx>("TestApp.HelloServer.HelloObj");
  ```

#### 单向调用

电向调用，表示客户端只管发送数据，而不接收服务端的响应，也不管服务端是否接收到请求

```java
HelloPrx prx = c.stringToProxy("TestApp.HelloServer.HelloObj");
//发起远程调用
prx.async_hello(null, 1000, "hello word");
```

#### 同步调用

```java
HelloPrx prx = c.stringToProxy("TestApp.HelloServer.HelloObj");
//发起远程调用
prx.hello(1000, "hello word");
```

#### 异步调用

```java
HelloPrx prx = c.stringToProxy("TestApp.HelloServer.HelloObj");
//发起远程调用
prx.async_hello(new HelloPrxCallback() {

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
}, 1000, "hello word");
```

> * 当接收到服务端返回时，HelloPrxCallback的callback_hello会被响应
> * 如果调用返回异常或超时，则callback_exception会被调用

