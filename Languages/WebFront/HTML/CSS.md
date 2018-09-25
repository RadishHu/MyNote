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

  