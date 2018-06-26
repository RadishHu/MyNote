# Vue模板语法

## 插值

- 文本

  使用`{{ }}`进行文本插值

  ```html
  <div id="app">
    <p>{{ message }}</p>
  </div>
  
  <script>
  new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue.js!'
    }
  })
  </script>
  ```

- Html

  使用v-html指令用于输出Html代码

  ```html
  <div id="app">
      <div v-html="message"></div>
  </div>
      
  <script>
  new Vue({
    el: '#app',
    data: {
      message: '<h1>鱼头飞天</h1>'
    }
  })
  </script>
  ```

- 属性

  Html属性中的值使用v-bind指令

  判断class1的值，如果为true使用class1类的样式，否则不使用该类：

  ```html
  <div id="app">
    <label for="r1">修改颜色</label><input type="checkbox" v-model="class1" id="r1">
    <br><br>
    <div v-bind:class="{'class1': class1}">
      鱼头飞天
    </div>
  </div>
      
  <script>
  new Vue({
      el: '#app',
    data:{
        class1: false
    }
  });
  </script>
  ```

- 表达式

  Vue提供完全的JavaScript表达式支持

  ```html
  <div id="app">
      {{5+5}}<br>
      {{ ok ? 'YES' : 'NO' }}<br>
      {{ message.split('').reverse().join('') }}
      <div v-bind:id="'list-' + id">鱼头飞天</div>
  </div>
      
  <script>
  new Vue({
    el: '#app',
    data: {
      ok: true,
      message: 'RUNOOB',
      id : 1
    }
  })
  </script>
  ```

## 指令

指令是带有`-v`前缀的

```html
<div id="app">
    <p v-if="seen">现在你看到我了</p>
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    seen: true
  }
})
</script>
```

v-if指令根据表达式seen值来决定是否插入p元素

- 参数

  参数在指令后以冒号指明：

  ```html
  <div id="app">
      <pre><a v-bind:href="url">菜鸟教程</a></pre>
  </div>
      
  <script>
  new Vue({
    el: '#app',
    data: {
      url: 'http://www.runoob.com'
    }
  })
  </script>
  ```

  v-bind指令将该元素的href属性与表示式url值绑定

- 修饰符

  修饰符是以半角句号.指明的特殊后缀：

  ```html
  <form v-on:submit.prevent="onSubmit"></form>
  ```

  .prevent修饰符告诉v-on指令对触发的事件调用event.preventDefault()

## 用户输入

使用v-model指令来实现双向数据绑定

```html
<div id="app">
    <p>{{ message }}</p>
    <input v-model="message">
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    message: 'Runoob!'
  }
})
</script>
```

## 过滤器

过滤器格式

```html
<!-- 在两个大括号中 -->
{{ message | capitalize }}

<!-- 在 v-bind 指令中 -->
<div v-bind:id="rawId | formatId"></div>
```

过滤器使用示例，将字符串第一个字母转为大写：

```html
<div id="app">
  {{ message | capitalize }}
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    message: 'runoob'
  },
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
})
</script>
```

## 缩写

- v-bind

  ```html
  <!-- 完整语法 -->
  <a v-bind:href="url"></a>
  <!-- 缩写 -->
  <a :href="url"></a>
  ```

- v-on

  ```html
  <!-- 完整语法 -->
  <a v-on:click="doSomething"></a>
  <!-- 缩写 -->
  <a @click="doSomething"></a>
  ```

  