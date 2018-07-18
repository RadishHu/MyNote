# Tars服务端开发

## 目录

> * [服务端开发](#chapter1)
> * [服务端配置](#chapter2)
> * [异步嵌套](#chapter3)

## 服务端开发 <a id="chapter1"></a>

### 配置依赖

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

### 接口文件定义

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

### 接口文件编译

提供插件编译生成java代码，在tars-maven-plugin添加生成java文件配置 :

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

执行maven命令`mvn tars:tars2java`，生成接口文件：

```java
@Servant
public interface HelloServant {

	public String hello(int no, String name);
}
```

### 服务接口实现

创建一个HelloServantImpl.java文件，实现HelloServant.java接口：

```java
public class HelloServantImpl implements HelloServant {

@Override
public String hello(int no, String name) {
    return String.format("hello no=%s, name=%s, time=%s", no, name, System.currentTimeMillis());
}
```

### 服务暴露

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

## 服务端开发配置 <a id="chapter2"></a>

### 服务配置ServerConfig

服务框架中的ServerConfig，记录了服务的基本信息，在服务框架初始化时，会自动从服务配置文件中初始化这些参数，配置文件格式如下：

```
<tars>
  <application>
    <server>
       #本地node的ip:port
       node=tars.tarsnode.ServerObj@tcp -h x.x.x.x -p 19386 -t 60000
       #应用名称
       app=TestApp
       #服务名称
       server=HelloServer
       #本机ip
       localip=x.x.x.x
       #管理端口
       local=tcp -h 127.0.0.1 -p 20001 -t 3000
       #服务可执行文件,配置文件等
       basepath=/usr/local/app/tars/tarsnode/data/TestApp.HelloServer/bin/
       #服务数据目录
       datapath=/usr/local/app/tars/tarsnode/data/TestApp.HelloServer/data/
       #日志路径
       logpath=/usr/local/app/tars/app_log/
       #配置中心的地址
       config=tars.tarsconfig.ConfigObj
       #通知上报的地址[可选]
       notify=tars.tarsnotify.NotifyObj
       #远程日志的地址[可选]
       log=tars.tarslog.LogObj
       #服务停止的超时时间
       deactivating-timeout=2000
       #日志等级
       logLevel=DEBUG
       #防空闲连接超时设置
	   sessionTimeOut=120000
	   #防空闲连接超时检查周期
       sessionCheckInterval=60000
    </server>
  </application>
</tars>
```

参数说明：

> Application：应用名称，如果配置文件没有配置，默认为UNKNOWN；
>
> ServerName：服务名称；
>
> BasePath：基本路径，通常表示可执行文件的路径；
>
> DataPath：数据文件路径，通常表示存在服务自己的数据；
>
> LocalIp：本地ip，默认是本机非127.0.0.1的第一块网卡IP；
>
> LogPath：日志文件路径，日志的写法请参考后续；
>
> LogLevel：滚动log日志级别；
>
> Local：服务可以有管理端口，可以通过管理端口发送命令给服务，该参数表示绑定的管理端口的地址，例如tcp -h 127.0.0.1 -p 8899，如果没有设置则没有管理端口；
>
> Node：本地NODE地址，如果设置，则定时给NODE发送心跳，否则不发送心跳，通常只有发布到框架上面的服务才有该参数；
>
> Log：日志中心地址，例如：tars.tarslog.LogObj@tcp –h .. –p …，如果没有配置，则不记录远程日志；
>
> Config：配置中心地址，例如：tars.tarsconfig.ConfigObj@tcp –h … -p …，如果没有配置，则addConfig函数无效，无法从远程配置中心拉取配置；
>
> Notify：信息上报中心地址，例如：tars.tarsnotify.NotifyObj@tcp –h … -p …，如果没有配置，则上报的信息直接丢弃；
>
> SessionTimeOut：防空闲连接超时设置；
>
> SessionCheckInterval：防空闲连接超时检查周期；

### Adapter

Adapter表示绑定端口，服务新增一个绑定端口，则新建立一个Adapter。

对于Tars服务而言，在服务配置中增加adapter项，即可以完成增加一个Servant处理对象

Adapter配置：

```
<tars>
  <application>
    <server>
       #配置绑定端口
       <TestApp.HelloServer.HelloObjAdapter>
            #允许的IP地址
            allow
            #监听IP地址
            endpoint=tcp -h x.x.x.x -p 20001 -t 60000
            #处理组
            handlegroup=TestApp.HelloServer.HelloObjAdapter
            #最大连接数
            maxconns=200000
            #协议
            protocol=tars
            #队列大小
            queuecap=10000
            #队列超时时间毫秒
            queuetimeout=60000
            #处理对象
            servant=TestApp.HelloServer.HelloObj
            #当前线程个数
            threads=5
       </TestApp.HelloServer.HelloObjAdapter>
    </server>
  </application>
</tars>
```

### 服务启动

服务启动时需要在启动命令中添加配置文件`-Dconfig=config.conf`，服务端和客户端的配置文件必须合并在这一个文件中。配置文件内容如下：

```
<tars>
  <application>
    enableset=n
    setdivision=NULL
    <server>
       #本地node的ip:port
       node=tars.tarsnode.ServerObj@tcp -h x.x.x.x -p 19386 -t 60000
       #应用名称
       app=TestApp
       #服务名称
       server=HelloServer
       #本机ip
       localip=x.x.x.x
       #管理端口
       local=tcp -h 127.0.0.1 -p 20001 -t 3000
       #服务可执行文件,配置文件等
       basepath=/usr/local/app/tars/tarsnode/data/TestApp.HelloServer/bin/
       #服务数据目录
       datapath=/usr/local/app/tars/tarsnode/data/TestApp.HelloServer/data/
       #日志路径
       logpath=/usr/local/app/tars/app_log/
       #配置中心的地址
       config=tars.tarsconfig.ConfigObj
       #通知上报的地址[可选]
       notify=tars.tarsnotify.NotifyObj
       #远程日志的地址[可选]
       log=tars.tarslog.LogObj
       #服务停止的超时时间
       deactivating-timeout=2000
       #日志等级
       logLevel=DEBUG
        #配置绑定端口
       <TestApp.HelloServer.HelloObjAdapter>
            #允许的IP地址
            allow
            #监听IP地址
            endpoint=tcp -h x.x.x.x -p 20001 -t 60000
            #处理组
            handlegroup=TestApp.HelloServer.HelloObjAdapter
            #最大连接数
            maxconns=200000
            #协议
            protocol=tars
            #队列大小
            queuecap=10000
            #队列超时时间毫秒
            queuetimeout=60000
            #处理对象
            servant=TestApp.HelloServer.HelloObj
            #当前线程个数
            threads=5
       </TestApp.HelloServer.HelloObjAdapter>
    </server>
    <client>
       #主控的地址
       locator=tars.tarsregistry.QueryObj@tcp -h x.x.x.x -p 17890
       #同步超时时间
       sync-invoke-timeout=3000
       #异步超时时间
       async-invoke-timeout=5000
       #刷新ip列表的时间间隔
       refresh-endpoint-interval=60000
       #上报数据的时间间隔
       report-interval=60000
       #采样率
       sample-rate=100000
       #最大采样数
       max-sample-count=50
       #模版名称
       modulename=TestApp.HelloServer
    </client>
  </application>

```

## 异步嵌套 <a id="chapter3"></a>

异步嵌套是指：

> A异步调用B，B接收到请求后再异步调用C，等C返回后，B再将结果返回A

通常情况下，B接收到请求后，在接口处理完毕后就需要返回应答给A，因此如果B在接口中又发起异步请求到C，则无法实现

解决方法：在实现接口方法中，声明启动异步来实现跨服务的异步调用

```java
//声明启动异步上下文
AsyncContext context = AsyncContext.startAsync();
//接口实现
...

//在异步处理后回包
context.writeResult(...);
```

## 服务命名 <a id="chapter4"></a>

Tars框架的服务，服务名称有三个部分：

- APP：应用名，表示一组服务的一个小集合，在Tars系统中，应用名必须唯一。例如：TestApp
- Server：服务名，提供服务的进程名称，Server名字根据业务服务功能命名，一般命名为：XXServer，例如：HelloServer
- Servant：服务者，提供具体服务的接口或实例，例如：HelloImp

一个Server可以包含多个Servant，系统会使用服务的App + Server + Servant 进行组合，来定义服务在系统中的路由名称，称为路由Obj，其名称在系统中必须是唯一的。例如：TestApp.HelloServer.HelloObj