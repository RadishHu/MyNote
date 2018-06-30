# Vue循环语句

循环使用v-for指令

- v-for

  ```html
  <div id="app">
    <ol>
      <li v-for="site in sites">
        {{ site.name }}
      </li>
    </ol>
  </div>
   
  <script>
  new Vue({
    el: '#app',
    data: {
      sites: [
        { name: 'Runoob' },
        { name: 'Google' },
        { name: 'Taobao' }
      ]
    }
  })
  </script>
  ```

- v-for迭代对象

  ```html
  <div id="app">
    <ul>
      <li v-for="value in object">
      {{ value }}
      </li>
    </ul>
  </div>
   
  <script>
  new Vue({
    el: '#app',
    data: {
      object: {
        name: '菜鸟教程',
        url: 'http://www.runoob.com',
        slogan: '学的不仅是技术，更是梦想！'
      }
    }
  })
  </script>
  ```

  - 第二个参数为键名：

    ```html
    <div id="app">
      <ul>
        <li v-for="(value, key) in object">
        {{ key }} : {{ value }}
        </li>
      </ul>
    </div>
    ```

  - 第三个参数为索引

    ```html
    <div id="app">
      <ul>
        <li v-for="(value, key, index) in object">
         {{ index }}. {{ key }} : {{ value }}
        </li>
      </ul>
    </div>
    ```

- v-for迭代整数

  ```html
  <div id="app">
    <ul>
      <li v-for="n in 10">
       {{ n }}
      </li>
    </ul>
  </div>
  ```