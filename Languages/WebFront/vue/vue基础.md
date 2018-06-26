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

