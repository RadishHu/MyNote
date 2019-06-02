# Vue基础

- 引入vue

  ```html
  <script src="./js/vue.min.js"></script>
  ```

- vue示例

  ```html
  <div id="vue_det">
      <h1>site:{{site}}</h1>
      <h1>url:{{url}}</h1>
      <h1>{{details()}}</h1>
  </div>
  <script type="text/javascript">
      var vm = new Vue({
          el:'#vue_det',
          data:{
              site:"鱼头飞天",
              url:"www.baidu.com",
              alexa:"1000"
          },
          methods:{
              details:function () {
                  return this.site + "- 我的，都是 我的";
              }
          }
      })
  </script>
  ```

  `data`用于定义属性

  `methods`用于定义函数，可以通过return返回函数值

  `{{ }}`用于输出对象属性和函数返回值

## 组件化应用

在 Vue 中组件本质上是一个拥有预定义选项的一个 Vue 实例

在 Vue 中注册组件：

```vue
// 定义名为 todo-item 的组件
Vue.component('todo-item',{
	template: '<li>这是个待办项</li>'
})
```

使用新注册的组件构建另一个组件模板：

```html
<ol>
    <!-- 创建一个 todo-item 组件的实例-->
    <todo-item></todo-item>
</ol>
```

如果要从父作用域将数据传到子组件，需要修改下组件：

```vue
Vue.component('todo-item', {
    props: ['todo'],
    template: '<li>{{todo.text}}</li>'
})

new Vue({
    el: '#app',
    data: {
        list: [
            {id: 0, text: '蔬菜'},
            {id: 1, text: '水果'},
            {id: 2, text: '肉'}
    	]
    }
})
```

使用 v-bind 指令将待办项传到循环输出的每个组件中：

```html
<div id="app">
    <ol>
        <todo-item
          v-for="item in list"
          v-bind:todo="item"
          v-bind:key="item.id"
         ></todo-item>
	</ol>
</div>
```

## Vue 实例

Vue 实例中会暴露一些实用的属性和方法，它们都有前缀 `$`，以便与用户定义的属性区分开

### Vue 实例生命周期钩子

Vue 实例在被创建时都要经过一系列的初始化过程，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM等。在这个过程中会运行一些叫做 `生命周期钩子` 的函数。

`created` 钩子可以用来在一个实例被创建之后执行代码：

```
new Vue({
    data: {
    a	: 1
    },
    created: function() {
    	console.log('a is: ' + this.a)
    }
})
```

还有一些其它钩子，在实例生命周期的不同阶段被调用哪个，如 `mounted` , `updated`, `destroyed`

### Vue 实例生命周期

1. init
2. compile template
3. mount
4. destroy





