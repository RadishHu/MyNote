# Vue条件语句

- v-if

  ```html
  <div id="app">
      <p v-if="seen">现在你看到我了</p>
      <template v-if="ok">
        <h1>菜鸟教程</h1>
        <p>学的不仅是技术，更是梦想！</p>
        <p>哈哈哈，打字辛苦啊！！！</p>
      </template>
  </div>
      
  <script>
  new Vue({
    el: '#app',
    data: {
      seen: true,
      ok: true
    }
  })
  </script>
  ```

  条件块可以这样写：

  ```html
  {{#if ok}}
    <h1>Yes</h1>
  {{/if}}
  ```

- v-else

  ```html
  <div id="app">
      <div v-if="Math.random() > 0.5">
        Sorry
      </div>
      <div v-else>
        Not sorry
      </div>
  </div>
      
  <script>
  new Vue({
    el: '#app'
  })
  </script>
  ```

- v-else-if

  ```html
  <div id="app">
      <div v-if="type === 'A'">
        A
      </div>
      <div v-else-if="type === 'B'">
        B
      </div>
      <div v-else-if="type === 'C'">
        C
      </div>
      <div v-else>
        Not A/B/C
      </div>
  </div>
      
  <script>
  new Vue({
    el: '#app',
    data: {
      type: 'C'
    }
  })
  </script>
  ```

- v-show

  ```html
  <body>
  <div id="app">
      <h1 v-show="ok">Hello!</h1>
  </div>
  	
  <script>
  new Vue({
    el: '#app',
    data: {
      ok: false
    }
  })
  </script>
  ```