# CSS

CSS，Cascading Style Sheets，用于渲染HTML元素标签的样式

下HTML中添加CSS的方式：

- 内联样式，在HTML元素中使用style属性

  ```html
  <p style="color:blue;margin-left:20px;">This is a paragraph.</p>
  ```

- 内部样式表，在HTML文档头部\<head>区域使用\<style>元素来包含CSS

  ```html
  <head>
  <style type="text/css">
  body {background-color:yellow;}
  p {color:blue;}
  </style>
  </head>
  ```

- 外部引用，使用CSS文件

  ```html
  <head>
  <link rel="stylesheet" type="text/css" href="mystyle.css">
  </head>
  ```

  

## margin 层叠样式表

margin 属性为给定元素设置四个方向 (上下左右) 的外边距属性，四个外边距属性分别是：`margin-top`, `margin-right`, `margin-bottom` 和 `margin-left`。

margin 的 top 和 bottom 属性对非替换内联元素无效，例如 `<span>` 和 `<code>`。

**语法：**

```css
margin: [<length> | <percentage> | auto]
```

> - \<length> 指定一个固定的宽度，可以为负数
> - \<percentage> 相对于该元素包含块的宽度，可以为负数
> - auto 浏览器会自动选择一个合适的 margin 来应用。可以用于将一个块居中

示例：

```css
margin: 1em;
margin: 5% auto;
margin: 1em auto 2em
margin: 2px 1em 0 auto
```

> - 一个值，这个值会被指定给四个边
> - 两个值，第一个值匹配上和下，第二个值匹配左和右
> - 三个值，第一个值匹配上，第二个值匹配左和右，第三个值匹配下
> - 四个值，依次按上、右、下、左的顺序匹配 (顺时针方向)