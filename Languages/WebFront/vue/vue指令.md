# vue 指令

## v-text

更新元素的 textContent

```html
<h1 v-text="msg"></h1>
```

## v-html

更新元素的 innerHtml

```html
<h1 v-html="msg"></h1>
```

## v-bind

绑定 html 属性，当表达式的值改变时，将其产生的连带影响，响应式的作用于 DOM

语法：

```
v-bind:title="msg"
```

简写：

```
:title="msg"
```

示例：

```html
<!-- 完整语法 -->
<a v-bind:href="url">test1</a>
<!-- 缩写 -->
<a :href="url">test2</a>

<script>
		new Vue({
			el: '#app',
			data: {
				url: 'http://www.baidu.com'
			}
		})
	</script>
<div id="app">
    <!-- 完整语法 -->
    <a v-bind:href="url">test1</a>
    <!-- 缩写 -->
    <a :href="url">test2</a>
</div>
<script>
    new Vue({
        el: '#app',
        data: {
            url: 'http://www.baidu.com'
        }
    })
</script>
```

## v-on

绑定事件

语法：

```
v-on:click="method"
```

简写：

```
@click="method"
```

> 绑定的事件写在 methods 属性中

示例：

```html
<div id="app">
    <!-- 完整写法 -->
    <a v-on:click="doSomething">test1</a>
    <!-- 简写 -->
    <a @click="doOtherThing('dodod')">test2</a>
</div>
<script>
    new Vue({
        el: '#app',
        methods: {
            doSomething: function() {
                alert('dododo')
            },
            doOtherThing: function() {
                alert('dodod')
            }
        }
    })
</script>
```

## v-model

在表单上创建双向数据绑定，监听用户的输入事件以更新数据

```html
<div id="app">
    <input v-model="message" placeholder="edit me"/>
    <p>Message is: {{message}}</p>
</div>
<script>
    new Vue({
        el: '#app',
        data: {
            message: ''
        }
    })
</script>
```

## v-for

基于源数据多次渲染元素或模板块

```html
<div id="app">
    <ul>
        <li v-for="item of list">{{item}}</li>
    </ul>
    <ul>
        <li v-for="(item,index) of list">{{item}}</li>
    </ul>
</div>
<script>
    new Vue({
        el: '#app',
        data: {
            list: [1,2,3]
        }
    })
</script>
```

