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





## 模板语法

Vue 使用了基于 HTML 的模板语法

## 插值

- 文本

  数据绑定常见的形式是使用"Mustache 语法"(双大括号)的文本插值：

  ```html
  <span>Message: {{ msg }}</span>
  ```

  > Mustache 标签会将被替换会对应数据对象上 `msg` 属性的值，当绑定的数据对象上 `msg` 属性发生改变，插值处的内容都会更新

- 原始 HTML

  `{{}}` 会将数据解释为普通文本，如果要输出 HTML，需要使用 `v-html` 指令

- 属性

  如果要把数据绑定到 HTML 属性上，可以使用 `v-bind` 指令：

  ```html
  <div v-bind:id="dynamicId"></div>
  ```

- 使用 JavaScript 表达式

  在绑定的数据中，Vue 提供了完全的 JavaScript 表达式支持

  ```html
  {{ number + 1 }}
  
  {{ ok ? 'YES' : 'NO' }}
  
  {{ message.split('').reverse().join('') }}
  
  <div v-bind:id="'list-' + id"></div>
  ```

  > Vue 只支持单个表达式

## 指令

指令的以 `v-` 作为前缀

- 参数

  一些指令可以接收一个参数，在指令之后以冒号表示：

  ```html
  <a v-bind:href="url">...</a>
  ```

  > v-bind 指令将该元素的 `href` 特性与表达式 `url` 的值绑定

- 动态参数

  可以用方括号括起来的 JavaScript 表达式作为一个指令的参数：

  ```html
  <a v-bind:[attributeName]="url"> ... </a>
  ```

  > attributeName 会被作为一个 JavaScript 表达式进行动态求值，求得的值将会作为最终的参数来使用。如果 Vue 实例有一个  data 属性 `attributeName`，其值为 `href`，那么这个绑定将等价于 `v-bind:href`

- 缩写

  Vue 为 `v-bind` 和 `v-on` 两个指令提供了特定的简写：

  v-bind 缩写

  ```html
  <!-- 完整语法 -->
  <a v-bind:href="url"></a>
  
  <!-- 缩写 -->
  <a :href="url"></a>
  ```

  v-on 缩写

  ```html
  <!-- 完整语法 -->
  <a v-on:click="doSomething">...</a>
  
  <!-- 缩写 -->
  <a @click="doSomething">...</a>
  ```

# Vue 属性

## computed 属性

```html
<body>
<div id="app">
	<p>Original message: "{{message}}"</p>
    <p>Computed reversed message: "{{reversedMessage}}"</p>
</div>
<script>
    new Vue({
    	el: '#app',
    	data: {
    		message: 'Hello Vue!'
    	},
        computed: {
            reversedMessage: function () {
                return this.message.split('').reverse().join('')
            }
        }
    })
</script>
</body>
```

> 这里声明了一个计算属性 `reversedMessage`，我们提供的函数将用作属性 `reversedMessage` 的 getter 函数

## methods 属性

在表达式中调用方法也可以达到 computed 属性同样的效果：

```html
<body>
<div id="app">
	<p>Original message: "{{message}}"</p>
    <p>Computed reversed message: "{{reversedMessage()}}"</p>
</div>
<script>
    new Vue({
    	el: '#app',
    	data: {
    		message: 'Hello Vue!'
    	},
        methods: {
            reversedMessage: function () {
                return this.message.split('').reverse().join('')
            }
        }
    })
</script>
</body>
```

`methods` 属性和 `computed` 属性最终实现的结果相同，不同之处在于：

- ` computed` 属性是基于它们的响应式依赖进行缓存的，只有在响应式依赖发生改变时它们才会重新求值，也就是说只要 `message` 没有发生变化，多次访问 `reversedMessage` computed属性会立即返回之前的结果，而不必再次执行函数。

- `methods` 属性每当触发重新渲染时，调用方法将总会再次执行函数

当我们有一个性能开销较大的 computed 属性 A，它需要遍历一个巨大的数组并做大量的计算，还要其它的 computed 属性依赖于 A。如果没有缓存，将会多次执行 A 的 getter。如果不希望有缓存，可以使用 methods 替代。

## watch 属性

侦听属性，用来实现一些数据随着其它数据变动而变动：

```html
<body>
    <div id="root">
        姓：<input v-model="firstName"/>
        名：<input v-model="lastName"/>
        <div>{{fullName}}</div>
        <div>{{count}}</div>
    </div>
    <script>
        new Vue({
            el: "#root",
            data: {
                firstName: '',
                lastName: '',
                count: 0
            },
            computed: {
                fullName: function() {
                    return this.firstName + ' ' + this.lastName
                }
            },
            watch: {
                firstName: function() {
                    this.count ++
                }
            }
        })
    </script>
</body>
```

> 当 firstName 值发生变化时，count 值也随之变化

# Class 和 Style 绑定

我们可以通过 `v-bind` 指令来对操作元素的 class 列表和内联样式进行数据绑定，除了通过表达式计算出字符串结果之外，还可以使用对象和数组

## 绑定 Html Class

- 对象语法

  可以传给 `v-bind:class` 一个对象，以动态地切换 class:

  ```html
  <div v-bind:class="{ active: isActive }"></div>
  ```

  > active 这个 class 存在与否取决于 data 属性 isActive 的 truthiness

  可以在对象中传入更多属性来动态切换多个 class:

  ```html
  <div class="static" v-bind:class="{ active: isActive, 'text-danger': hasError}"></div>
  
  data: {
  	isActive: true,
  	hasError: false
  }
  ```

  > v-bind:class 指令可以与普通的 class 属性共存

  渲染结果为：

  ```html
  <div class="static active"></div>
  ```

  绑定的数据对象也可以定义在 data 属性中：

  ```html
  <div v-bind:class="classObject"></div>
  
  data: {
    classObject: {
      active: true,
      'text-danger': false
    }
  }
  ```

  渲染效果为：

  ```html
  <div class="static active"></div>
  ```

  也可以绑定一个返回对象的计算属性 ：

  ```html
  <div v-bind:class="classObject"></div>
  
  data: {
    isActive: true,
    error: null
  },
  computed: {
    classObject: function () {
      return {
        active: this.isActive && !this.error,
        'text-danger': this.error && this.error.type === 'fatal'
      }
    }
  }
  ```

- 数组语法

  可以把一个数组传给 v-bind:class，以应用一个 class 列表：

  ```html
  <div v-bind:class="[activeClass, errorClass]"></div>
  
  data: {
    activeClass: 'active',
    errorClass: 'text-danger'
  }
  ```

  渲染为：

  ```html
  <div class="active text-danger"></div>
  ```

  通过三元表达式，可以根据条件切换列表中的 class:

  ```html
  <div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
  ```

  > 这样写始终会有 errorClass，只有在 isActive 为 truthy 时才添加 activeClass

  当有多个条件 class 时这样写会比较繁琐，可以在数组语法中使用对象语法：

  ```html
  <div v-bind:class="[{active: isActive}, errorClass]"></div>
  ```

- 用在组件上

  当在一个自定义组件上使用 class 属性时，这些类将被添加到该组件的根元素上，根元素上已有的类不会被覆盖：

  ```js
  Vue.component('my-component',{
      templete: '<p class="foo bar">Hi</p>'
  })
  ```

  在使用该组件时添加class:

  ```html
  <my-component class="baz boo"></my-component>
  ```

  渲染为：

  ```html
  <p class="foo bar baz boo">Hi</p>
  ```

## 绑定 Style

# 条件渲染

## v-if

`v-if` 指令用于 条件性地渲染一块内容，这块内容只会在指令的表达式返回 truthy 值的时候被渲染。

```html
<h1 v-if="awesome">Vue is awesome</h1>
```

可以用 `v-else` 添加一个 else 块：

```html
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>others</h1>
```

> v-else 元素必须紧跟在带 v-if 或 v-else-if 的元素后面，否则将不会被识别

可以使用 `v-else-if` 添加 else-if 块

## \<template> 元素使用 v-if

v-if 必须添加到一个元素上，如果想切换多个元素，可以把 \<template> 元素当做不可见的包裹元素，并在上使用 v-if，最终的渲染结果将不包含 \<template> 元素

```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

## v-show

另一个用于根据条件展示元素的选项是 `v-show` 指令：

```html
<h1 v-show="ok">Hello!</h1>
```

> v-show 通过切换元素的 CSS 属性 `display` 来控制元素是否展示
>
> v-show 不支持 \<template> 元素，也不支持 v-else

## v-if VS v-show

v-if 确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建

v-if 是惰性的，如果在初始渲染时条件为假，则不会渲染条件快，直到条件块变为真

v-show 不管初始条件是什么，元素总是会被渲染，并只是基于 CSS 进行切换

v-if 有更高的切换开销，v-show 有更高的初始渲染开销。如果需要非常频繁的切换，则使用 v-show; 如果在运行时条件很少改变，则使用 v-if 较好

# 列表渲染

## v-for 数组

语法：

```
v-for="item in items"

# 带索引
v-for="(item, index) in items"

# 使用 of 代替 in
v-for="item of items"
```

> items 是源数据数组
>
> item 是数组元素迭代的别名

```html
<ul id="app">
    <li v-for="item in items">
    	{{item.message}}
    </li>
</ul>
```

```js
var example = new Vue({
    el: "#app",
    data: {
        items: [
            {message: 'Foo'},
            {message: 'Bar'}
        ]
    }
})
```

在使用 v-for 时提供 key 属性：

```html
<div v-for="item in items" v-bind:key="item.id">
    <!-- 内容 -->
</div>
```

## v-for 对象

语法：

```
v-for="value in object"

# 带键名
v-for="(value, name) in object"

# 带索引
v-for="(value, name, index) in object"
```

> 遍历对象是按 `Object.keys()` 的结果遍历

```html
<ul id="app">
    <li v-for="value in object"
        {{value}}
    </li>
</ul>
```

```js
new Vue({
    el: "#app",
    data: {
        object: {
            title: 'How to do lists in Vue',
            author: 'Jane Doe',
            publishedAt: '2016-04-10'
        }
    }
})
```

