# Maven入门

## 目录

> * [Maven特点](#chapter1)
> * [Maven项目的目录结构](#chapter2)
> * [Maven仓库](#chapter3)
> * [Maven环境配置](#chapter4)

## Maven特点 <a id="chapter1"></a>

Maven是apache下的开源项目, 项目管理工具,管理java项目

- 项目对象模型(POM,Project Object Model)

  项目对象模型,每个maven工程中都有一个pom.xml文件,定义工程所依赖的jar包,本工程的坐标,打包运行的方式

- 依赖管理系统

  maven通过坐标对项目工程所依赖的jar包统一规范管理

- maven定义了一套项目生命周期

  清理, 初始化, 编译, 测试, 报告, 打包, 部署, 站点生成

- maven统一开发规范

  maven对项目的目录结构做了统一的规划, 源代码, 单元测试代码, 配置文件,资源文件 都放在响应的目录下

## Maven项目的目录结构 <a id="chapter2"></a>

---| src

------------| main				存放项目的主要代码

------------------------| java		存放java代码

------------------------| resources	存放配置文件

------------------------| webapp		存放web应用相关代码

------------| test				存放项目测试相关的代码

---| pom.xml					maven项目核心配置文件

---| target					存放编译输出后的代码

## Maven仓库 <a id="chapter3"></a>

Maven仓库用来存放Maven管理的所有jar包,分为: 本地仓库 和远程仓库.远程仓库分为:中央仓库和私服, 当项目编译时,Maven首先从本地仓库中寻找项目所需的jar包, 若本地仓库没有,在到Maven的中央仓库下载

maven默认的中央仓库为: http://repo1.maven.org/maven2

寻找jar包:http://mvnrepository.com

## Maven环境配置 <a id="chapter4"></a>

常用的开发工具Idea,Eclipse里都已经集成了Maven,不过一般还是从官网下载配置到电脑里

Maven官网: <http://maven.apache.org/>

Maven官网下载: http://maven.apache.org/download.cgi

Maven历史版本下载: <https://archive.apache.org/dist/maven/binaries/>

- 下载完解压,配置环境变量,跟JDK的环境变量配置方法类似:

  ```
  MAVEN_HOME
  F:\Develop\apache-maven-3.2.1
  Path
  %MAVEN_HOME%/bin;
  ```

  验证:win+r ---> cmd ---> 输入 mvn --version

- 修改maven配置

  配置文件: %Maven文件位置%\conf\settings.xml

  配置本地仓库路径的位置:

  ```
  <localRepository>F:\\Develop\Maven_Repository</localRepository>
  ```

  配置mirror镜像站点:

  ```xml
  <mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
  </mirror>
  ```

  把远程仓库的地址改为alibaba的镜像, 把原来的镜像注释掉,添加这个配置

