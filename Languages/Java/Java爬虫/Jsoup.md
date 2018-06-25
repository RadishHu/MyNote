# Jsoup

pom依赖

```xml
<dependency>
    <!-- jsoup HTML parser library @ https://jsoup.org/ -->
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.10.3</version>
</dependency>
```

## 文档输入

jsoup可以从字符串、URL地址、本地文件来加载HTML文档，并生成Document对象实例：

- 从字符串加载Html文档

  ```java
  String html = "<html><head><title> 开源中国社区 </title></head>"
    + "<body><p> 这里是 jsoup 项目的相关文章 </p></body></html>"; 
  Document doc = Jsoup.parse(html);
  ```

- 从URL加载Html

  ```java
  Document doc = Jsoup.connect("http://www.oschina.net/").get(); 
  String title = doc.title(); 
  
  Document doc = Jsoup.connect("http://www.oschina.net/") 
      .data("query", "Java")   // 请求参数
      .userAgent("I ’ m jsoup") // 设置 User-Agent 
      .cookie("auth", "token") // 设置 cookie 
      .timeout(3000)           // 设置连接超时时间
      .post();                 // 使用 POST 方法访问 URL 
  ```

- 从文件加载Html文档

  ```java
  File input = new File("D:/test.html"); 
  Document doc = Jsoup.parse(input,"UTF-8","http://www.oschina.net/");
  ```


## 选择器

基本用法：

| 选择器                                    | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| tagname                                   | 使用标签名定位，例如a                                        |
| ns\|tag                                   | 使用命名空间的标签定位，例如fb:name用来查找\<fb:name>元素    |
| \#id                                      | 使用元素id定位                                               |
| .class                                    | 使用元素的class属性定位                                      |
| [attribute]                               | 使用元素的属性进行定位，例如[href]表示检索具有href属性的所有元素 |
| [^attr]                                   | 使用元素属性名前缀进行定位                                   |
| [attr=value]                              | 使用属性值进行定位，例如[width=500]定位所有width属性值为500的元素 |
| [attr^=value],[attr$=value],[attr*=value] | 分别代表属性以value开头、结尾以及包含                        |
| [attr~=regex]                             | 使用正则表达式进行属性值的过滤                               |
| *                                         | 定位所有元素                                                 |

组合用法：

| 选择器               | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| el#id                | 定位id为指定值的元素，例如a#logo -> \<div class=head>xxx\</dev> |
| el.class             | 定位class为指定值的元素                                      |
| el[attr]             | 定位属性值为指定值的元素                                     |
| 以上三种可以任意组合 | a[href]#logo 、a[name].outerlink                             |
| parent > child       |                                                              |
| siblingA + aiblingB  |                                                              |
| siblingA ~ siblingX  |                                                              |
| el,el,el             |                                                              |
|                      |                                                              |

表达式：

| 表达式             | 说明                                                |
| ------------------ | --------------------------------------------------- |
| :lt(n)             | td:lt(3)表示小于三列的表格                          |
| :gt(n)             | div p:gt(2)表示div中包含2个以上的p                  |
| :eq(n)             | form input:eq(1)表示只包含一个input的表单           |
| :has(selector)     | div:has(p)表示包含p元素的div                        |
| :not(selector)     | div:not(.logo)表示不包含class=logo元素的所有div列表 |
| :contains(text)    | 包含指定文本的元素，不区分大小写                    |
| :containsOwn(text) | 文本信息完全等于指定条件的过滤                      |
| :matches(regex)    | 使用正则表达式进行文本过滤                          |
| :matchesOwn(regex) |                                                     |

