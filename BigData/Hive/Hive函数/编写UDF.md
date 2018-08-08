# 编写UDF

编写一个UDF，需要继承UDF类并实现`evaluate()`函数。在查询执行过程中，查询中的每个应用到这个函数的地方都会对这个类进行实例化。对于每行输入都会调用`evaluate()`函数。

## UDF项目构建

```xml
<properties>
    <project.build.sourceEncoding>UTF8</project.build.sourceEncoding>
    <!--Hadoop版本更改成自己的版本-->
    <hadoop.version>2.6.0-cdh5.7.0</hadoop.version>
    <hive.version>1.1.0-cdh5.7.0</hive.version>
</properties>

<repositories>
    <!--加入Hadoop原生态的maven仓库的地址-->
    <repository>
        <id>Apache Hadoop</id>
        <name>Apache Hadoop</name>
        <url>https://repo1.maven.org/maven2/</url>
    </repository>
    <!--加入cdh的maven仓库的地址-->
    <repository>
        <id>cloudera</id>
        <name>cloudera</name>
        <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    </repository>
</repositories>

<dependencies>
    <!--添加hadoop依赖-->
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>${hadoop.version}</version>
    </dependency>
    <!--添加hive依赖-->
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
        <version>${hive.version}</version>
    </dependency>
```

## 通过日期计算出星座的UDF

- 定义UDF

```java
package com.yutou.udf;

import org.apache.hadoop.hive.ql.exec.Description;
import org.apache.hadoop.hive.ql.exec.UDF;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

@Description(
        name = "zodiac",
        value = "_FUNC_(date)-from the input data string" +
                "or separate month and day arguments,return the sign of Zodiac.",
        extended = "Example:\n" +
                "> SELECT _FUNC_(date_string) FROM src;\n" +
                "> SELECT _FUNC_(month,day) FROM src;"
)

public class UDFZodiacSign extends UDF {

    private SimpleDateFormat df;

    public UDFZodiacSign() {
        df = new SimpleDateFormat("MM-dd-yyyy");
    }

    public String evaluate(Date date) {
        return evaluate(date.getMonth() + 1,date.getDate());
    }

    public String evaluate(String body) {
        Date date = null;
        try {
            date = df.parse(body);
        } catch (ParseException e) {
            return null;
        }
        return evaluate(date.getMonth() + 1,date.getDate());
    }

    public String evaluate(Integer month,Integer day) {
        if (month == 1) {
            if (day < 20) {
                return "Capricorn";
            } else {
                return "Aquarius";
            }
        }
        if (month == 2) {
            if (day < 19) {
                return "Aquarius";
            } else {
                return "Pisces";
            }
        }
        // and so on
        return null;
    }
    
}
```

> * @Description()是java的注解，可选。注解中注明了关于这个函数的文档说明，可以通过这个注解描述UDF的使用方法和例子。这样通过`DESCRIBE FUNCTION`命令查看函数时，注解中的\_FUNC_字符串会被替换为用户为这个函数定义的临时函数名称
> * evaluate()函数的参数和返回值类型只能是Hive可序列化的数据类型，例如：如果要处理数值，UDF的输出参数类型可以是基本数据类型int、Integer封装的对象或者IntWritable对象。当类型不一致时，Hive会自动将类型转换为匹配的类型
> * null在Hive中对任何数据类型都是合法的，但是对于Java基本数据类型，不能是对象，也不能是null

- 将编译后的UDF二进制文件打包成一个JAR文件

- 将JAR文件加入到类路径下：

  ```
  hive> ADD JAR /full/path/to/zodiac.jar
  ```

- 通过`CREATE FUNCTION`语句定义这个java类的函数：

  ```
  hive> CREATE TEMPORARY FUNCTION zodiac
  ```

  > TEMPORARY关键字，当前会话中声明的函数只会在当前会话中有效，用户需要在每个会话中都增加JAR，然后创建函数。如果用户需要频繁地使用同一个JAR文件和函数，可以将相关语句增加到`$HOME/.hiverc`文件中

- 删除函数的方法：

  ```
  hive> DROP TEMPORARY FUNCTION IF EXISTS zodiac;
  ```

