# Hive安装

## 安装环境准备

- Hive安装前需要安装好JDK和Hadoop，并配置好环境变量

- 安装MySQL

  Hive元数据管理有两种方式：

  内置derby：

  - 解压hive安装包
  - bin/hive启动即可使用
  - 缺点：运行hive会在当前目录生成一个derby文件和metastore_db目录，用户不可以执行2个并发的Hive CLI实例

  MySQL：

  - 解压，修改配置文件hive-site.xml
  - 配置mysql元数据库信息

## 安装

- 下载解压

  ```http
  https://mirrors.tuna.tsinghua.edu.cn/apache/hive/
  ```

- 修改配置文件

  - hive-env.sh

    配置$hadoop_home

  - hive-site.xml

    ```xml
    <configuration>
    <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
    <description>JDBC connect string for a JDBC metastore</description>
    </property>
    
    <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
    </property>
    
    <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
    <description>username to use against metastore database</description>
    </property>
    
    <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>root</value>
    <description>password to use against metastore database</description>
    </property>
    </configuration>
    ```

- 拷贝jar包

  下载mysql-connector-java-xxx.jar，拷贝到$HIVE_HOME/lib目录下

- Jline包版本不一致问题

  拷贝hive的lib目录中jline.2.12.jar的jar包替换/home/hadoop/app/hadoop-2.6.4/share/hadoop/yarn/lib/jline-0.9.94.jar

# Hive启动方式

- Hive交互式shell

  ```
  bin/hive
  ```

- hive启动为一个服务，对外提供服务

  ```
  bin/hiveserver2
  #或后台启动
  nohup bin/hiveserver2 1>/var/log/hiveserver.log 2>/var/log/hiveserver.err &
  ```

  启动成功后，可以在别的节点上用beeline连接

  ```
  bin/beeline -u jdbc:hive://host:10000 -n -root
  #或者
  bin/beeline
  ```

- 提交hive命令，执行完后返回linux界面

  ```
  hive -e 'sql'
  ```