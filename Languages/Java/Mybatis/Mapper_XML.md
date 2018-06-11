# Mapper XML文件

SQL 映射文件顶级元素（按照它们应该被定义的顺序）： 

- chache：命名空间的缓存配置 
- cache-ref：引用其他命名空间缓存配置 
- resultMap：用来描述如何从数据库结果集中加载对象 
- sql：可被其他语句引用的可重用语句块 
- insert：映射插入语句 
- update：映射更新语句 
- delete：映射删除语句 
- select：映射查询语句 

## select

- 查询语句

  ```xml
  <select id="selectPerson" parameterType="int" resultType="hashmap">
    SELECT * FROM PERSON WHERE ID = #{id}
  </select>
  ```

  符号`#{id}`，告诉 MyBatis 创建一个预处理语句参数，通过 JDBC，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中：

  ```java
  // Similar JDBC code, NOT MyBatis…
  String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
  PreparedStatement ps = conn.prepareStatement(selectPerson);
  ps.setInt(1,id);
  ```

- select元素的属性配置

  ```xml
  <select
    id="selectPerson"
    parameterType="int"
    parameterMap="deprecated"
    resultType="hashmap"
    resultMap="personResultMap"
    flushCache="false"
    useCache="true"
    timeout="10000"
    fetchSize="256"
    statementType="PREPARED"
    resultSetType="FORWARD_ONLY">
  ```

  | 属性          | 描述                                                         |
  | ------------- | ------------------------------------------------------------ |
  | id            | 命名空间中唯一的标识符，可以被用来引用这条语句               |
  | parameterType | 将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset |
  | parameterMap  | 已经被废弃                                                   |
  | resultType    | 从这条语句中返回的期望类型的类的完全限定名或别名。注意如果是集合情形，那应该是集合可以包含的类型，而不能是集合本身 。resultType和resultMap不能同时使用 |
  | resultMap     | 外部 resultMap 的命名引用                                    |
  | flushCache    | 将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false |
  | useCache      | 将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true |
  | timeout       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动） |
  | fetchSize     | 这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动） |
  | statementType | STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED |
  | resultSetType | FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动） |
  | databaseId    | 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略 |
  | resultOrdered | 这个设置仅针对嵌套结果 select 语句适用：如果为 true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至于导致内存不够用。默认值：`false` |
  | resultSets    | 这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的 |

## sql

- sql元素可以被用来定义可重用的SQL代码，可以包含在其他语句中

  ```xml
  <sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
  ```

  这个SQL片段可以被包含在其他语句中：

  ```xml
  <select id="selectUsers" resultType="map">
    select
      <include refid="userColumns"><property name="alias" value="t1"/></include>,
      <include refid="userColumns"><property name="alias" value="t2"/></include>
    from some_table t1
      cross join some_table t2
  </select>
  ```

  示例：

  ```xml
  <sql id="sometable">
    ${prefix}Table
  </sql>
  
  <sql id="someinclude">
    from
      <include refid="${include_target}"/>
  </sql>
  
  <select id="select" resultType="map">
    select
      field1, field2, field3
    <include refid="someinclude">
      <property name="prefix" value="Some"/>
      <property name="include_target" value="sometable"/>
    </include>
  </select>
  ```

## ResultMaps





## 类型别名

使用类型别名不用输入类的完全限定名称：

```xml
<typeAlias type="com.someapp.model.User" alias="User"/>
```

