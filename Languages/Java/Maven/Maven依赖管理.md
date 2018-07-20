# Maven依赖管理

## 目录

> * [pom.xml文件](#"chapter1")
> * [jar包依赖范围](#chapter2)
> * [依赖传递](#chapter3)

## pom.xml文件 <a id="chapter1"></a>

- modelVersion

  指定当前Maven模型的版本, 对于Maven2和Maven3,这个版本是4.0.0

- groupId

  这个一般是公司或组织的名字, 一般由三部分组成:

  第一部分是项目的性质, 商业性质的是 com,非盈利性组织的是 org;

  第二部分是公司名,apache,alibaba;

  第三部分是项目名,dubbo,zookeeper;

- aritifactId

  Maven构建的项目名

- version

  版本号, SNAPSHOT快照的意思, 表明项目还在开发中,是不稳定版本

  groupId,artifactId,version构成了一个Maven项目的基本坐标

- packaging

  项目的打包类型: jar,war,rar,ear,pom, 默认的是jar

- dependencies, dependency

  项目依赖的一些jar包,就使用dependency引入

- properties

  用来定义一些配置属性,例如项目构建源码编码方式,可以设置为UTF-8,防止中文乱码; 也可以定义相关构建版本号, 便于版本的统一管理

- build

  build表示与构建相关的配置文件

## jar包依赖范围 <a id="chapter2"></a>

scope用来定义依赖的jar包的作用范围：

| 依赖范围(Scope) | 对主代码有效 | 对测试代码有效 | 打包后,运行时有效 |
| :-------------: | :----------: | :------------: | :---------------: |
|     compile     |      Y       |       Y        |         Y         |
|    provided     |      Y       |       Y        |         -         |
|      test       |      -       |       Y        |         -         |
|     runtime     |      -       |       -        |         Y         |

scope的默认值为：compile

## 依赖传递 <a id="chapter3"></a>

如果项目引用了一个Jar包,而该Jar包引用了其它Jar包,那么默认情况下项目编译时,Maven会把直接和间接引用的Jar包都下载到本地

makeFriend ---> helloFriend ---> hello	makeFriend依赖helloFriend, 也会依赖hello

依赖控制: 控制自己的依赖的包是否往下传递

```xml
<dependency>
  <groupId>cn.yutou.hello</groupId>
  <artifactId>hello</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <scope>compile</scope>
  <!-- 表示依赖的hello是否往下传递
  如果是true，表示不往下传递,只作用于当前的包;如果是false，表示继续往下传递
  -->
  <optional>true</optional>
</dependency>
```

依赖排除:排除掉某些不需要的jar包

```xml
<dependency>
  <groupId>cn.itcast.helloFriend</groupId>
  <artifactId>helloFriend</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <!-- 通过  exclusions  来排除我们不需要的一些依赖的jar包-->
  <exclusions>
    <exclusion>
    <groupId>cn.itcast.hello</groupId>
    <artifactId>hello</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

