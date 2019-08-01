# CSS 盒子模型

盒子模型，Box Model，它包括：外边距，边框，内边距和实际内容

![盒子模型](https://www.runoob.com/images/box-model.gif)

- Margin (外边距) - 边框外的区域，外边距是透明的
- Border (边框) - 围绕在内边距和内容外的边框
- Padding(内边距) - 内容周围的区域，用于呈现元素的北京，内边距是透明的
- Content(内容) - 显示的实际内容

## 元素的宽度和高度

总元素的宽度 = 宽度 + 左填充 + 右填充 + 左边框 + 右边框 + 左边距 + 右边距

总元素的高度 = 高度 + 顶部填充 + 底部填充 + 上边框 + 下边框 + 上边距 + 下边距

```html
div {
	width: 300px;
	border: 25px solid green;
	padding: 25px;
	margin: 25px;
}
```

> 300px(宽) + 50px(左 + 右填充) + 50px(左 + 右边框) + 50px(左 + 右边距) = 450px

