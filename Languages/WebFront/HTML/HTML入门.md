# HTML入门

## 目录

> - [<!DOCTYPE>声明](#chapter1)
> - HTML标签
>   - [HTML标题](#chapter2-1)
>   - [HTML段落](#chapter2-1)
>   - [HTML链接](#chapter2-3)
>   - [HTML图像](#chapter2-4)
>   - [HTML表格](#chapter2-5)
>   - [HTML列表](#chapter2-6)
>   - [HTML区块](#chapter2-7)
>   - [其它标签](#chapter2-8)
> - [HTML属性](#chapter3)
> - [HTML \<head>元素](#chapter4)
> - [HTML表单](#chapter5)
> - [HTML框架](#chapter6)
> - [HTML颜色](#chapter7)
> - [HTML脚本](#chapter8)
> - [HTML URL](#chapter10)

HTML，Hyper Text Markup Language，超文本标记语言

HTML不是一种编程语言，而是一种标记语言

HTML通过标记标签来描述网页

## <!DOCTYPE>声明 <a id="chapter1"></a>

<!DOCTYPE>声明用于帮助浏览器正确显示网页

doctype生命不区分大小写

通用声明：

- HTML5

  ```html
  <!DOCTYPE html>
  ```

- HTML4.01

  ```html
  <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
  "http://www.w3.org/TR/html4/loose.dtd">
  ```

- XHTML1.0

  ```html
  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
  ```

## HTML标签

### HTML标题 <a id="chapter2-1"></a>

HTML标题通过\<h1>-\<h6>标签来定义：

```html
<h1>这是一个标题</h1>
<h2>这是一个标题</h2>
<h3>这是一个标题</h3>
```

### HTML段落 <a id="chater2-2"></a>

HTML段落通过标签\<p>来定义：

```html
<p>这是一个段落。</p>
<p>这是另外一个段落。</p>
```

### HTML链接 <a id="chater2-3"></a>

HTML链接是通过标签<a>来定义：

```html
<a href="http://www.runoob.com">这是一个链接</a>
```

### HTML图像 <a id="chapter2-4"></a>

HTML图像是通过标签\<img>来定义：

```html
<img src="/images/logo.png" width="258" height="39" />
```

- alt属性：

  alt属性用来为图像定义一串预备的可替换的文本，在浏览器无法载入图像时，浏览器会显示替换性的文本而不是图像

  ```html
  <img src="boat.gif" alt="Big Boat">
  ```

- 设置图像的高度与宽度

  height与width属性用于设置图像的高度与宽度

  ```html
  <img src="pulpit.jpg" alt="Pulpit rock" width="304" height="228">
  ```

### HTML表格 <a id="chapter2-5"></a>

表格由\<table>标签来定义，每个表格有若干行(通过\<tr>标签定义)，每行被分割为若干单元格(由\<td>标签定义，table data)。单元格可以包含文本、图片、列表、段落、表单、水平线、表格等

```html
<table border="1">
    <tr>
        <td>row 1, cell 1</td>
        <td>row 1, cell 2</td>
    </tr>
    <tr>
        <td>row 2, cell 1</td>
        <td>row 2, cell 2</td>
    </tr>
</table>
```

- 边框属性

  使用边框属性来显示一个带有边框的表格，如果不定义边框属性，表格将不显示边框

  ```html
  <table border="1">
      <tr>
          <td>Row 1, cell 1</td>
          <td>Row 1, cell 2</td>
      </tr>
  </table>
  ```

- 表头

  表格的表头使用\<th>标签定义，大多数浏览器会把表头显示为粗体居中的文本：

  ```html
  <table border="1">
      <tr>
          <th>Header 1</th>
          <th>Header 2</th>
      </tr>
      <tr>
          <td>row 1, cell 1</td>
          <td>row 1, cell 2</td>
      </tr>
      <tr>
          <td>row 2, cell 1</td>
          <td>row 2, cell 2</td>
      </tr>
  </table>
  ```

### HTML列表 < a id="chapter2-6"></a>

HTML支持有序列表、无序列表和定义列表

- 无序列表

  无序列表使用\<ul>标签，每项使用粗体圆点进行标记

  ```html
  <ul>
  <li>Coffee</li>
  <li>Milk</li>
  </ul>
  ```

- 有序列表

  有序列表使用\<ol>标签，列表的项目使用数字进行标记

  ```html
  <ol>
  <li>Coffee</li>
  <li>Milk</li>
  </ol>
  ```

- 自定义列表

  自定义列表使用\<dl>标签，自定义列表的项目包括项目和注释。每个自定义列表项以\<dt>开始，注释以\<dd>开始

  ```html
  <dl>
  <dt>Coffee</dt>
  <dd>- black hot drink</dd>
  <dt>Milk</dt>
  <dd>- white cold drink</dd>
  </dl>
  ```

### HTML区块 <a id="chapter2-7"></a>

大多数HTML元素被定义为块级元素和内联元素：

块级元素在浏览器显示时，通常会以新行来开始

内联元素在显示时通常不会以新行开始

- \<div>元素

  \<div>元素是块级元素，它可以用于组合其它HTML元素

  \<div>没有特定含义，如果与CSS一同使用，\<div>元素可用于对大的内容块设置样式属性

  \<div>元素的另一个常见的用途是文档布局，它取代了使用表格定义布局的老式方法

- \<span>元素

  \<span>元素是内联元素，可用作文本的容器

  \<span>元素没有特定的含义，当与CSS一同使用时，\<span>元素可用于为部分文本设定样式属性

### 其它标签 <a id="chapter2-8"></a>

- \<hr>

  \<hr>标签在HTML页面创建水平线

- 注释

  <!-- 注释内容 -->

- \<br>

  \<br>标签用于换行

## HTML属性 <a id="chapter3"></a>

HTML标签元素可以设置属性

属性可以在元素中添加附加信息

属性一般在开始标签中

属性总是以名称/值的形式出现，name=“value”

属性值始终包括在引号内，双引号、单引号都可以

适用于大多数HTML元素的属性：

- class，为html元素定义一个或多个类名，类名从样式文件引入
- id，定义元素唯一id
- style，规定元素的行内样式
- title，描述元素的额外信息

## HTML \<head>元素 <a id="chapter4"></a>

\<head>元素包含了所有的头部标签，在<head>元素中国可以插入脚本(scripts)，样式文件(CSS)，以及各种meta信息

可以在头部添加的元素标签有：\<title>, \<style>, \<meta>, \<link>, \<script>, \<noscript>, \<base>

- \<title>元素

  \<title>标签定义了不同文档的标题

  \<title>在HTML/XHTML文档中是必须的

  \<title>元素：

  ​	定义了浏览器工具栏的标题

  ​	当网页添加到收藏夹时，显示在收藏夹中的标题

  ​	显示在搜索引擎结果页面的标题

  ```html
  <!DOCTYPE html>
  <html>
  <head> 
  <meta charset="utf-8"> 
  <title>文档标题</title>
  </head>
   
  <body>
  文档内容......
  </body>
   
  </html>
  ```

- \<base>元素

  \<base>标签描述了基本的链接地址/链接目标，该标签作为HTML文档中所有链接标签的默认链接

  ```html
  <head>
  <base href="http://www.runoob.com/images/" target="_blank">
  </head>
  ```

- \<link>元素

  \<link>标签定义了文档与外部资源之间的关系，通常用于链接到样式表

  ```html
  <head>
  <link rel="stylesheet" type="text/css" href="mystyle.css">
  </head>
  ```

- \<style>元素

  \<style>标签可以直接添加样式渲染HTML文档

  ```html
  <head>
  <style type="text/css">
  body {background-color:yellow}
  p {color:blue}
  </style>
  </head>
  ```

- \<meta>元素

  \<meta>标签提供了元数据，元数据不会显示在页面上，但是会被浏览器解析

  META元素通常用于制定网页的描述，关键词，文件的最后修改时间，作者，和其它元数据：

  为搜索引擎定义关键词：

  ```html
  <meta name="keywords" content="HTML, CSS, XML, XHTML, JavaScript">
  ```

  为网页定义描述内容：

  ```html
  <meta name="description" content="免费 Web & 编程 教程">
  ```

  定义网页作者：

  ```html
  <meta name="author" content="Runoob">
  ```

  每30秒刷新当前页面：

  ```html
  <meta http-equiv="refresh" content="30">
  ```

- \<script>元素

  \<script>标签用于加载脚本文件，如JavaScript

## HTML表单 <a id="chapter5"></a>

HTML表单用于收集不同类型的用户输入

表单允许用户在表单中输入内容，比如：文本域(textarea)、下拉列表、单选框(radio-buttons)、复选框(checkboxes)等

表单使用表单元素\<form>来设置：

```html
<form>
.
input 元素
.
</form>
```

用的比较多的表单标签是输入标签`<input>`

输入类型是由类型属性`type`定义，常用的输入类型有：

- 文本域

  文本域通过\<input type="text">标签类设定，当用户要在表单中输入字母、数字等内容时，就会用到文本域

  ```html
  <form>
  First name: <input type="text" name="firstname"><br>
  Last name: <input type="text" name="lastname">
  </form>
  ```

  > 大多数浏览器中，文本域的缺省宽度为20个字符

- 密码字段

  密码字段通过\<input type="password">标签来设定

  ```html
  <form>
  Password: <input type="password" name="pwd">
  </form>
  ```

- 单选框

  单选框通过\<input type="radio">标签来设定

  ```html
  <form>
  <input type="radio" name="sex" value="male">Male<br>
  <input type="radio" name="sex" value="female">Female
  </form>
  ```

- 复选框

  复选框通过\<input type="checkbox">标签来设定

  ```html
  <form>
  <input type="checkbox" name="vehicle" value="Bike">I have a bike<br>
  <input type="checkbox" name="vehicle" value="Car">I have a car 
  </form>
  ```

- 提交按钮

  提交按钮通过\<input type="submit">标签来定义

  当用户点击提交按钮时，表单的内容会被传送到另一个文件，表单的动作属性定义了目的文件的文件名

  ```html
  <form name="input" action="html_form_action.php" method="get">
  Username: <input type="text" name="user">
  <input type="submit" value="Submit">
  </form>
  ```

## HTML框架 <a id="chapter6"></a>

通过使用框架，可以在同一个浏览器窗口中显示不止一个页面

iframe语法：

```html
<iframe src="URL"></iframe>
```

>  该URL指向不同的网页

- iframe设置高度与宽度

  height和width属性用来定义iframe标签的高度和宽度

  该属性默认以像素为单位，也可以制定按比例显示

  ```html
  <iframe src="demo_iframe.htm" width="200" height="200"></iframe>
  ```

- iframe移除边框

  frameborder属性用于定义iframe是否显示边框

  设置属性值为0，移除iframe的边框

  ```html
  <iframe src="demo_iframe.htm" frameborder="0"></iframe>
  ```

- 使用iframe显示目标链接页面

  ```html
  <iframe src="demo_iframe.htm" name="iframe_a"></iframe>
  <p><a href="http://www.runoob.com" target="iframe_a">RUNOOB.COM</a></p>
  ```

  > a标签的target属性是名为iframe_a的iframe框架，所有在点击a标签链接时，页面会显示在iframe框架中

## HTML颜色 <a id="chapter7"></a>

HTML颜色由一个十六进制符号来定义，这个符号由红色、绿色和蓝色的值组成(RGB)

每种颜色的最小值是0(十六进制：#00)，最大值是255(十六进制：#FF)

## HTML脚本 <a id="chapter8"></a>

JavaScript使HTML页面具有更强的动态和交互性

\<script>标签用于定义客户端脚本，既可以包含脚本语句，也可以通过src属性指向外部脚本文件

JavaScript最常用于图片操作、表单验证以及内容动态更新

```html
<script>
document.write("Hello World!");
</script>
```

\<noscript>标签提供无法使用脚本时的替代内容，它可以包含普通HTML页面的body元素中能够找到的所有元素

只要在浏览器不支持脚本或者禁用脚本时，才会显示\<noscript>元素中的内容

```html
<script>
document.write("Hello World!")
</script>
<noscript>抱歉，你的浏览器不支持 JavaScript!</noscript>
```

## HTML字符实体 <a id="chapter9"></a>

在HTML中，一些字符是预留的，比如，不能直接使用小于号(<)和大于号(>)，因为浏览器会误认为它们是标签

如果希望正确地显示预留字符，必须在HTML源代码中使用字符实体，如需显示小于号，可以这样写：`&lt`、`&#60`或者`&#060`

## HTML URL <a id="chapter10"></a>

URL，Uniform Resource Locators，统一资源定位器

URL语法规则：

```
scheme://host.domain:port/path/filename
```

> - scheme，定义因特网服务的类型，最常见的类型是http
> - host，定义域主机，http默认主机是www
> - domain，定义因特网域名，比如baidu.com
> - :port，定义主机上的端口号，http默认端口为80
> - path，定义服务器上的路径
> - filename，定义文档/资源的名称

URL使用`ASCII`字符集

URL常常会包含ASCII集合之外的字符，URL必须转换为有效的ASCII格式

URL编码使用`%`其后跟随两位的十六进制数来替换非ASCII字符

URL不能包含空格，通常使用+来替换空格