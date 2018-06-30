# Vue-resource

## 引入vue-resource

```html
<script src="js/vue.js"></script>
<script src="js/vue-resource.js"></script>
```

## 基本语法

- 基于全局Vue对象使用http

  ```js
  Vue.http.get('/someUrl',[options]).then(successCallback,errorCallback);
  Vue.http.post('someUrl',[body],[options]).then(successCallback,errorCallback);
  ```

- 在一个Vue实例内使用$http

  ```js
  this.$http.get('/someUrl',[options]).then(successCallback,errorCallback);
  this.$http.post('/someUrl',[body],[options]).then(successCallback,errorCallback);
  ```

  发送请求后，使用`then`方法处理响应结果，then方法有两个参数：

  - 第一个参数响应成功时的回调函数
  - 第二个参数响应失败时的回调函数

  回调函数有两种写法：

  - 传统函数写法：

    ```js
    this.$http.get('/someUrl',[options]).then(
        function(response){
            //响应成功回调
        }，function(response){
        	//响应错误回调
    	}
    )；
    ```

  - Lamda写法

    ```js
    this.$http.get('/someUrl',[options]).then(
        (response)=>{
            //响应成功回调
        }，
        （response)=>{
        	//响应错误回调
    	}
    )
    ```

## 支持的Http方法

- get(url,[options])
- head(url,[options])
- delete(url,[options])
- jsoup(url,[options])
- post(url,[body],[options])
- put(url,[body],[options])
- patch(url,[body],[options])

## options对象

发送请求时options对象可以包含的属性：

| 参数        | 类型                   | 描述                                                         |
| ----------- | ---------------------- | ------------------------------------------------------------ |
| url         | string                 | 请求的URL                                                    |
| method      | string                 | 请求的HTTP方法，例如：‘GET','POST'或其它HTTP方法             |
| body        | Object,FormData,string | request body                                                 |
| params      | Object                 | 请求的URL参数对象                                            |
| headers     | Object                 | request header                                               |
| timeout     | number                 | 请求超时时间，单位毫秒，0表示无超时时间                      |
| before      | function(request)      | 请求发送前的处理函数，类似于jQuery的beforeSend函数           |
| progress    | function(event)        | ProgressEvent回调处理函数                                    |
| credentials | boolean                | 表示跨域请求时是否需要使用凭证                               |
| emulateHTTP | boolean                | 发送PUT,PATCH,DELETE请求时以HTTP POST的方式发送，并设置请求头的X-HTTP-Method-Override |
| emulateJSON | boolean                | 将request body以application/x-www-form-urlencoded content type发送 |

## response对象

response对象包含的方法：

| 方法   | 类型   | 描述                            |
| ------ | ------ | ------------------------------- |
| text() | string | 以String形式返回response body   |
| json() | Object | 以JSON对象形式返回response body |
| blob() | Blob   | 以二进制形式返回response body   |

response对象包含的属性：

| 属性       | 类型    | 描述                                          |
| ---------- | ------- | --------------------------------------------- |
| ok         | boolean | 响应的HTTP状态码在200~299之间时，该属性为true |
| status     | number  | 响应的HTTP状态码                              |
| statusText | string  | 响应的状态文本                                |
| headers    | Object  | 响应头                                        |

## 发送请求示例

- GET请求

  ```js
  var demo = new Vue({
      el: '#app',
      data: {
          gridColumns: ['customerId', 'companyName', 'contactName', 'phone'],
          gridData: [],
          apiUrl: 'http://211.149.193.19:8080/api/customers'
      },
      ready: function() {
          this.getCustomers()
      },
      methods: {
          getCustomers: function() {
              this.$http.get(this.apiUrl)
                  .then((response) => {
                      this.$set('gridData', response.data)
                  })
                  .catch(function(response) {
                      console.log(response)
                  })
          }
      }
  })
  ```

- POST请求

  ```js
  var demo = new Vue({
      el: '#app',
      data: {
          show: false,
          gridColumns: [{
              name: 'customerId',
              isKey: true
          }, {
              name: 'companyName'
          }, {
              name: 'contactName'
          }, {
              name: 'phone'
          }],
          gridData: [],
          apiUrl: 'http://211.149.193.19:8080/api/customers',
          item: {}
      },
      ready: function() {
          this.getCustomers()
      },
      methods: {
          closeDialog: function() {
              this.show = false
          },
          getCustomers: function() {
              var vm = this
              vm.$http.get(vm.apiUrl)
                  .then((response) => {
                      vm.$set('gridData', response.data)
                  })
          },
          createCustomer: function() {
              var vm = this
              vm.$http.post(vm.apiUrl, vm.item)
                  .then((response) => {
                      vm.$set('item', {})
                      vm.getCustomers()
                  })
              this.show = false
          }
      }
  })
  ```

  