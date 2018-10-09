# HTML5

## 目录

> * [HTML5 Input类型](#chapter1)
> * [HTML5表单元素](#chapter2)
> * [HTML5Web存储](#chapter3)

HTML5新特性：

- 用于绘画的canvas元素
- 用于媒介回放的video和audio元素
- 对本地离线存储更好的支持，本地SQL数据
- 新的特殊内容元素：article、footer、header、nav、section等
- 新的表单控件：calendar、date、time、email、url、search等

## HTML5 Input类型 <a id="chapter1"></a>

HTML5拥有一些新的表单输入类型：color、date、datetime、datetime-local、email、month、number、range、search、tel、time、url、week

### color

color类型用在input字段，主要用来选取颜色

```html
选择你喜欢的颜色: <input type="color" name="favcolor">
```

### date

date类型允许从一个日期选择器选择一个日期

```html
生日: <input type="date" name="bday">
```

### datetime

datetime类型允许选择一个日期(UTC时间)

```html
生日 (日期和时间): <input type="datetime" name="bdaytime">
```

### datetime-local

datetime-local类型允许选择一个日期和时间(无时区)

```html
生日 (日期和时间): <input type="datetime-local" name="bdaytime">
```

### email

email类型用于包含email地址的输入域：

```html
E-mail: <input type="email" name="email">
```

### month

month类型允许选择一个月份

```html
生日 (月和年): <input type="month" name="bdaymonth">
```

### number

number类型用于应该包含数值的输入域

```html
数量 ( 1 到 5 之间 ): <input type="number" name="quantity" min="1" max="5">
```

### range

rane类型用于应该包含一定范围内数字值的输入域

range类型显示为滑动条

```html
<input type="range" name="points" min="1" max="10">
```

### search

search类型用于搜索域，比如百度搜索

```html
Search Google: <input type="search" name="googlesearch">
```

### tel

定义输入电话号码字段

```html
电话号码: <input type="tel" name="usrtel">
```

### time

time类型允许选择一个时间

```html
选择时间: <input type="time" name="usr_time">
```

### url

url类型用于应该包含url地址的输入域

在提交表单时，会自动验证url域的值

```html
添加您的主页: <input type="url" name="homepage">
```

### week

week类型允许选择周和年

```html
选择周: <input type="week" name="week_year">
```

## HTML5表单元素 <a id="chapter2"><a/>

HTML5有一下新的表单元素：\<datalist>、\<keygen>、\<output>

### \<datalist>元素

\<datalist>规定输入域的选项列表

\<datalist>属性规定form或input域应该拥有自动完成功能，当用户在自动完成域中开始输入时，浏览器应该在该域中显示填写的选项

使用\<input>元素的列表属性与\<datalist>元素绑定：

```html
<input list="browsers">
 
<datalist id="browsers">
  <option value="Internet Explorer">
  <option value="Firefox">
  <option value="Chrome">
  <option value="Opera">
  <option value="Safari">
</datalist>
```

### \<keygen>元素

\<keygen>元素的作用是提供一种验证用户的可靠方法

\<keygen>标签规定用于表单的密钥对生成器字段

当提交表单时，会生成两个键：一个私钥，一个公钥

私钥存储于客户端，公钥则被发送到服务器，公钥可用于之后验证用户的客户端证书

```html
<form action="demo_keygen.asp" method="get">
用户名: <input type="text" name="usr_name">
加密: <keygen name="security">
<input type="submit">
</form>
```

### \<output>元素

\<output>元素用于不同类型的输出

```html
<form oninput="x.value=parseInt(a.value)+parseInt(b.value)">0
<input type="range" id="a" value="50">100 +
<input type="number" id="b" value="50">=
<output name="x" for="a b"></output>
</form>
```

## HTML5Web存储 <a id="chapter3"></a>

以前本地存储使用的是cookie，但是Web存储需要更加安全与快速，这些数据不会保存在服务器上，但是这些数据只用于请求网站数据上，它也可以存储大量的数据

数据以键/值对存储，web网页的数据只允许该网页访问使用

客户端存储数据的两个对象为：

- localStorage

  用于长久保存整个网站的数据，保存的数据没有过期时间，直到手动去除

- sessionStorage

  用于临时保存同一窗口(或标签页)的数据，在关闭窗口或标签页之后将会删除这些数据