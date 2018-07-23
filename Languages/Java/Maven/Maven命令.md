# Maven命令

通过命令创建maven java工程

```
mvn archetype:generate 
	-DgroupId=cn.itcast.maven.quickstart 
	-DartifactId=simple 
	-DarchetypeArtifactId=maven-archetype-quickstart 
	-DarchetypeCatalog=internal
```

maven常用命令

- 打包,将项目打包成jar包 或者 war包

  ```
  mvn package
  ```

- 清理

  ```
  mvn clean
  ```

- 编译命令

  ```
  mvn compile
  ```

- 将项目安装到本地仓库

  ```
  mvn install
  ```

## 参数说明

