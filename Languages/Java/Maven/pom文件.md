# pom.xml文件

## 目录

>* [基础配置](#chapter1)
>* [构建配置](#chapter2)
>* [profile配置](#chapter3)

## 基础配置 <a id="chapter1"></a>

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- 指定当前Maven模型的版本, 对于Maven2和Maven3,这个版本是4.0.0 -->
    <modelVersion>4.0.0</modelVersion>

    <!-- 这个一般是公司或组织的名字, 一般由三部分组成:
  	第一部分是项目的性质, 商业性质的是 com,非盈利性组织的是 org;
    第二部分是公司名,apache,alibaba;
    第三部分是项目名,dubbo,zookeeper;
   -->
    <groupId>com.winner.trade</groupId>

    <!-- 项目名 -->
    <artifactId>projectName</artifactId>

    <!-- 版本号,groupId,artifactId,version构成了一个Maven项目的基本坐标-->
    <version>1.0.0-SNAPSHOT</version>

    <!-- 打包的机制，如pom,jar, maven-plugin, ejb, war, ear, rar, par，默认为jar -->
    <packaging>jar</packaging>

    <!-- 非必填 -->
    <name>projectName</name>
    <description>projectDesc</description>

    <!-- 帮助定义构件输出的一些附属构件,附属构件与主构件对应，有时候需要加上classifier才能唯一的确定该构件 不能直接定义项目的classifer,因为附属构件不是项目直接默认生成的，而是由附加的插件帮助生成的 -->
    <classifier>...</classifier>

    <!-- 定义本项目的依赖关系 -->
    <dependencies>

        <!-- 每个dependency都对应这一个jar包 -->
        <dependency>

            <!--maven通过groupId、artifactId、version这三个元素值来检索jar包， 然后引入工程 -->
            <groupId>com.winner.trade</groupId>
            <artifactId>trade-test</artifactId>
            <version>1.0.0-SNAPSHOT</version>

            <!-- jar包的作用范围 -->
            <scope>test</scope>

            <!-- 依赖控制，默认false，表示继续往下传递；true，表示不往下传递，只作用于当前包 -->
            <optional>false</optional>

            <!-- 依赖排除，排除不需要的依赖的jar包 -->
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>

        </dependency>

    </dependencies>

    <!-- 为pom定义一些常量，在pom中的其它地方可以直接引用 使用方式 如下 ：${file.encoding} -->
    <properties>
        <file.encoding>UTF-8</file.encoding>
        <java.source.version>1.5</java.source.version>
        <java.target.version>1.5</java.target.version>
    </properties>

    <!-- 构建配置 -->
    <build></build>
    
    <!-- profile配合 -->
    <profiles></profiles>


    ...
</project>
```

## 构建配置 <a id="chapter2"></a>

```xml
<build>

    <!-- 产生的构件的文件名，默认值是${artifactId}-${version}。 -->
    <finalName>porjectName</finalName>

    <!-- 构建产生的所有文件存放的目录,默认为${basedir}/target，即项目根目录下的target -->
    <directory>${basedir}/target</directory>

    <!--当项目没有规定目标（Maven2叫做阶段（phase））时的默认值， -->
    <!--必须跟命令行上的参数相同例如jar:jar，或者与某个阶段（phase）相同例如install、compile等 -->
    <defaultGoal>install</defaultGoal>

    <!--当filtering开关打开时，使用到的过滤器属性文件列表。 -->
    <!--项目配置信息中诸如${spring.version}之类的占位符会被属性文件中的实际值替换掉 -->
    <filters>
        <filter>../filter.properties</filter>
    </filters>

    <!--项目相关的所有资源路径列表，例如和项目相关的配置文件、属性文件，这些资源被包含在最终的打包文件里。 -->
    <resources>
        <resource>

            <!--资源的目标路径,该路径相对target/classes目录） 
			   举个例子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为org/apache/maven/messages
			-->
            <targetPath>resources</targetPath>

            <!--是否使用参数值代替参数名。参数值取自properties元素或者文件里配置的属性，文件在filters元素里列出。 -->
            <filtering>true</filtering>

            <!--描述存放资源的目录，该路径相对POM路径 -->
            <directory>src/main/resources</directory>

            <!--包含的模式列表 -->
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>

            <!--排除的模式列表 如果<include>与<exclude>划定的范围存在冲突，以<exclude>为准 -->
            <excludes>
                <exclude>jdbc.properties</exclude>
            </excludes>

        </resource>
    </resources>

    <!--单元测试相关的所有资源路径，配制方法与resources类似 -->
    <testResources>
        <testResource>
            <targetPath />
            <filtering />
            <directory />
            <includes />
            <excludes />
        </testResource>
    </testResources>

    <!--项目源码目录，当构建项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。 -->
    <sourceDirectory>${basedir}\src\main\java</sourceDirectory>

    <!--项目脚本源码目录，该目录和源码目录不同，绝大多数情况下，该目录下的内容会被拷贝到输出目录(因为脚本是被解释的，而不是被编译的)。 -->
    <scriptSourceDirectory>${basedir}\src\main\scripts</scriptSourceDirectory>

    <!--项目单元测试使用的源码目录，当测试项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。 -->
    <testSourceDirectory>${basedir}\src\test\java</testSourceDirectory>

    <!--被编译过的应用程序class文件存放的目录。 -->
    <outputDirectory>${basedir}\target\classes</outputDirectory>

    <!--被编译过的测试class文件存放的目录。 -->
    <testOutputDirectory>${basedir}\target\test-classes</testOutputDirectory>

    <!--项目的一系列构建扩展,它们是一系列build过程中要使用的产品，会包含在running bulid‘s classpath里面。 -->
    <!--他们可以开启extensions，也可以通过提供条件来激活plugins。 -->
    <!--简单来讲，extensions是在build过程被激活的产品 -->
    <extensions>

        <!--例如，通常情况下，程序开发完成后部署到线上Linux服务器，可能需要经历打包、 -->
        <!--将包文件传到服务器、SSH连上服务器、敲命令启动程序等一系列繁琐的步骤。 -->
        <!--实际上这些步骤都可以通过Maven的一个插件 wagon-maven-plugin 来自动完成 -->
        <!--下面的扩展插件wagon-ssh用于通过SSH的方式连接远程服务器， -->
        <!--类似的还有支持ftp方式的wagon-ftp插件 -->
        <extension>
            <groupId>org.apache.maven.wagon</groupId>
            <artifactId>wagon-ssh</artifactId>
            <version>2.8</version>
        </extension>

    </extensions>

    <!--使用的插件列表 。 -->
    <plugins>
        <plugin>
            <groupId></groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.5.5</version>

            <!--在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。 -->
            <executions>
                <execution>

                    <!--执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标 -->
                    <id>assembly</id>

                    <!--绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段 -->
                    <phase>package</phase>

                    <!--配置的执行目标 -->
                    <goals>
                        <goal>single</goal>
                    </goals>

                    <!--配置是否被传播到子POM -->
                    <inherited>false</inherited>

                </execution>
            </executions>

            <!--作为DOM对象的配置,配置项因插件而异 -->
            <configuration>
                <finalName>${finalName}</finalName>
                <appendAssemblyId>false</appendAssemblyId>
                <descriptor>assembly.xml</descriptor>
            </configuration>

            <!--是否从该插件下载Maven扩展（例如打包和类型处理器）， -->
            <!--由于性能原因，只有在真需要下载时，该元素才被设置成true。 -->
            <extensions>false</extensions>

            <!--项目引入插件所需要的额外依赖 -->
            <dependencies>
                <dependency>...</dependency>
            </dependencies>

            <!--任何配置是否被传播到子项目 -->
            <inherited>true</inherited>

        </plugin>
    </plugins>

    <!--主要定义插件的共同元素、扩展元素集合，类似于dependencyManagement， -->
    <!--所有继承于此项目的子项目都能使用。该插件配置项直到被引用时才会被解析或绑定到生命周期。 -->
    <!--给定插件的任何本地配置都会覆盖这里的配置 -->
    <pluginManagement>
        <plugins>...</plugins>
    </pluginManagement>

</build>
```

## profile配置 <a id="chapter3"></a>

```xml
<profiles>

    <!-- 根据环境参数或命令行参数激活某个构建处理 -->
    <profile>
        <!-- 自动触发profile的条件 -->
        <activation>

            <!-- profile默认是否激活 -->
            <activeByDefault>false</activeByDefault>

            <!-- activation内建的java版本检测,如果检测到jdk版本与期待的一样,profile被激活 -->
            <jdk>1.7</jdk>

            <!--os元素定义操作系统相关的属性,当匹配的操作系统属性被检测到,profile被激活 -->
            <os>

                <!--激活profile的操作系统的名字 -->
                <name>Windows XP</name>

                <!--激活profile的操作系统所属家族(如 'windows') -->
                <family>Windows</family>

                <!--激活profile的操作系统体系结构 -->
                <arch>x86</arch>

                <!--激活profile的操作系统版本 -->
                <version>5.1.2600</version>

            </os>

            <!--如果Maven检测到某一个属性(其值可以在POM中通过${名称}引用),其拥有对应的名称和值,Profile就会被激活
			如果值字段是空的,那么存在属性名称字段就会激活profile,否则按区分大小写方式匹配属性值字段
			-->
            <property>

                <!--激活profile的属性的名称 -->
                <name>mavenVersion</name>

                <!--激活profile的属性的值 -->
                <value>2.0.3</value>

            </property>

            <!--提供一个文件名,通过检测该文件的存在或不存在来激活profile
			missing检查文件是否存在,如果不存在则激活profile
			exists则会检查文件是否存在,如果存在则激活profile
			-->
            <file>

                <!--如果指定的文件存在,则激活profile。 -->
                <exists>/usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/</exists>

                <!--如果指定的文件不存在,则激活profile。 -->
                <missing>/usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/</missing>

            </file>

        </activation>
        <id />
        <build />
        <modules />
        <repositories />
        <pluginRepositories />
        <dependencies />
        <reporting />
        <dependencyManagement />
        <distributionManagement />
        <properties />
    </profile>
```

