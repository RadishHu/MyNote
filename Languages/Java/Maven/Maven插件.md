# Maven插件

maven插件都包裹在`<build></build>` 标签中的`<plugins></plugins>`标签中

## Tomcat

```xml
<plugin>
  <groupId>org.apache.tomcat.maven</groupId>
  <artifactId>tomcat7-maven-plugin</artifactId>
  <configuration>
    <path>/</path>
    <port>8086</port>
  </configuration>
</plugin>
```

## 指定JDK版本

```xml
<plugin>  
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-compiler-plugin</artifactId>  
  <version>3.1</version>  
  <configuration>  
    <source>1.8</source>  
    <target>1.8</target> 
    <encoding>utf-8</encoding> 
  </configuration>  
</plugin>
```

## 源码打包插件

```xml
<plugin>  
  <artifactId>maven-source-plugin</artifactId>  
  <version>2.4</version>  
  <configuration>  
    <attach>true</attach>  
  </configuration>  
  <executions>  
    <execution>  
      <phase>package</phase>  
      <goals>  
      	<goal>jar-no-fork</goal>  
      </goals>  
    </execution>  
  </executions>  
</plugin>  

<plugin>
  <artifactId> maven-assembly-plugin </artifactId>
  <configuration>
    <descriptorRefs>
      <descriptorRef>jar-with-dependencies</descriptorRef>
    </descriptorRefs>
    <archive>
      <manifest>
      <mainClass>com.cetc.di.App</mainClass>
      </manifest>
    </archive>
  </configuration>
  <executions>
    <execution>
    <id>make-assembly</id>
    <phase>package</phase>
    <goals>
    	<goal>single</goal>
    </goals>
    </execution>
  </executions>
</plugin>
```

