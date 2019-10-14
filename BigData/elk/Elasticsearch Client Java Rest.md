# 概述

Java Rest Client 有两种：

- 低级 Rest Client：通过 **http** 跟 Elasticsearch 连接，它跟所有的 Elasticsearch 版本兼容。
- 高级 Rest Client：基于低级 client，暴露一些方法

# 低级 Rest Client

### Maven Repository

- Mavent configuration

  ```xml
  <dependency>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>elasticsearch-rest-client</artifactId>
      <version>7.1.1</version>
  </dependency>
  ```

- Gradle configuration

  ```gradle
  dependencies {
      compile 'org.elasticsearch.client:elasticsearch-rest-client:7.1.1'
  }
  ```

### Shading

- Maven configuration

  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-shade-plugin</artifactId>
              <version>3.1.0</version>
              <executions>
                  <execution>
                      <phase>package</phase>
                      <goals><goal>shade</goal></goals>
                      <configuration>
                          <relocations>
                              <relocation>
                                  <pattern>org.apache.http</pattern>
                                  <shadedPattern>hidden.org.apache.http</shadedPattern>
                              </relocation>
                              <relocation>
                                  <pattern>org.apache.logging</pattern>
                                  <shadedPattern>hidden.org.apache.logging</shadedPattern>
                              </relocation>
                              <relocation>
                                  <pattern>org.apache.commons.codec</pattern>
                                  <shadedPattern>hidden.org.apache.commons.codec</shadedPattern>
                              </relocation>
                              <relocation>
                                  <pattern>org.apache.commons.logging</pattern>
                                  <shadedPattern>hidden.org.apache.commons.logging</shadedPattern>
                              </relocation>
                          </relocations>
                      </configuration>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>
  ```

- Gradle configuration

  ```gradle
  shadowJar {
      relocate 'org.apache.http', 'hidden.org.apache.http'
      relocate 'org.apache.logging', 'hidden.org.apache.logging'
      relocate 'org.apache.commons.codec', 'hidden.org.apache.commons.codec'
      relocate 'org.apache.commons.logging', 'hidden.org.apache.commons.logging'
  }
  ```

### 初始化

`RestClient` 实例可以通过对应的 `RestClientBuilder` 来创建，通过 `RestClient.builder(HttpHost...)` 方法：

```java
RestClient restClient = RestClient.builder(
    new HttpHost("localhost", 9200, "http"),
    new HttpHost("localhost", 9201, "http")).build();
```

`RestClient` 是线程安全的，跟调用它的程序有相同的生命周期，使用完的时候需要手动的关闭它，以便释放它所使用的所有资源、http 客户端和线程等：

```java
restClient.close();
```

也可以通过 `RestClientBuilder` 设置配置参数：

```java
RestClientBuilder builder = RestClient.builder(
    new HttpHost("localhost", 9200, "http"));
Header[] defaultHeaders = new Header[]{new BasicHeader("header", "value")};
builder.setDefaultHeaders(defaultHeaders);
```

> 设置每个请求的请求头，这样就不用再每个请求上单独设置

```shell
RestClientBuilder builder = RestClient.builder(
        new HttpHost("localhost", 9200, "http"));
builder.setFailureListener(new RestClient.FailureListener() {
    @Override
    public void onFailure(Node node) {
        // content
    }
});
```

> 设置节点失败监听，content 定义失败时执行的操作。只有当开启 **sniffing on failure** 时才可以使用。

```java
RestClientBuilder builder = RestClient.builder(
    new HttpHost("localhost", 9200, "http"));
builder.setNodeSelector(NodeSelector.SKIP_DEDICATED_MASTERS);
```

> 设置节点选择器，通过过滤客户端发送请求的节点。用来防止把请求发送到专用的节点上，默认情况下，请求会发送到所有配置的节点上。

```java
RestClientBuilder builder = RestClient.builder(
        new HttpHost("localhost", 9200, "http"));
builder.setRequestConfigCallback(
    new RestClientBuilder.RequestConfigCallback() {
        @Override
        public RequestConfig.Builder customizeRequestConfig(
                RequestConfig.Builder requestConfigBuilder) {
            return requestConfigBuilder.setSocketTimeout(10000); 
        }
    });
```

> 设置一个回调函数，修改默认的请求配置 (比如：请求超时，验证，或者所有 [org.apache.http.client.config.RequestConfig.Builder](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/client/config/RequestConfig.Builder.html) 允许设置的参数)

```java
RestClientBuilder builder = RestClient.builder(
    new HttpHost("localhost", 9200, "http"));
builder.setHttpClientConfigCallback(new HttpClientConfigCallback() {
        @Override
        public HttpAsyncClientBuilder customizeHttpClient(
                HttpAsyncClientBuilder httpClientBuilder) {
            return httpClientBuilder.setProxy(
                new HttpHost("proxy", 9000, "http"));  
        }
    });
```

> 设置一个回调函数，修改 http client 配置 (比如：通过 ssl 加密，或者所有 [`org.apache.http.impl.nio.client.HttpAsyncClientBuilder`](http://hc.apache.org/httpcomponents-asyncclient-dev/httpasyncclient/apidocs/org/apache/http/impl/nio/client/HttpAsyncClientBuilder.html) 允许设置的参数)

### 发送请求

可以通过 `performRequest` 或 `performRequestAsync` 来发送请求：

- performRequest

  发送同步请求，发送请求时会锁住调用的线程，返回的结果是 `Response` 对象

  ```java
  Request request = new Request(
      "GET",  
      "/");   
  Response response = restClient.performRequest(request);
  ```

  > 可以使用所有的 HTTP 方法：GET, POST, HEAD 等

- performRequestAsync

  ```java
  Request request = new Request(
      "GET",  
      "/");   
  restClient.performRequestAsync(request, new ResponseListener() {
      @Override
      public void onSuccess(Response response) {
          
      }
  
      @Override
      public void onFailure(Exception exception) {
          
      }
  });
  ```

  > 可以使用所有的 HTTP 方法：GET, POST, HEAD 等
  >
  > onSuccess 用来处理请求返回的结果
  >
  > onFailure 用来出以请求异常结果

给请求添加参数：

```java
request.addParameter("pretty", "true");
```

给 `HttpEntity` 设置请求体：

```java
request.setEntity(new NStringEntity(
        "{\"json\":\"text\"}",
        ContentType.APPLICATION_JSON));
```

> 给 HttpEntity 设置 ContentType 是非常重要的，因为它会设置 Content-Type，Elasticsearch 会根据这个对数据进行转换

也可以通过下面的方式设置 `ContentType` 为 `application/json`：

```java
request.setJsonEntity("{\"json\":\"text\"}");
```

**RequestOptions**

`RequestOptions` 类负责一个客户端程序中所有请求的共享部分，可以只创建一个 `RequestOptions` 示例在所有的请求中共享：

```java
private static final RequestOptions COMMON_OPTIONS;
static {
    RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();
    builder.addHeader("Authorization", "Bearer " + TOKEN); 
    builder.setHttpAsyncResponseConsumerFactory(           
        new HeapBufferedResponseConsumerFactory(30 * 1024 * 1024 * 1024));
    COMMON_OPTIONS = builder.build();
}
```

> builder.addHeader()，添加一个所有请求都需要的请求头
>
> builder.setHttpAsyncResponseConsumerFactory()，设置 response consumer 在 JVM 堆中保存信息的大小

创建 RequestOptions 后可以在 request 中添加：

```java
request.setOptions(COMMON_OPTIONS);
```

也可以给每个 request 单独设置 RequestOptions:

```java
RequestOptions.Builder options = COMMON_OPTIONS.toBuilder();
options.addHeader("cats", "knock things off of other things");
request.setOptions(options);
```

**并行的发送异步请求**

这个示例说明并行的 index 多个文档，但是在实际应用中会使用 `_bulk` API：

```java
final CountDownLatch latch = new CountDownLatch(documents.length);
for (int i = 0; i < documents.length; i++) {
    Request request = new Request("PUT", "/posts/doc/" + i);
    //let's assume that the documents are stored in an HttpEntity array
    request.setEntity(documents[i]);
    restClient.performRequestAsync(
            request,
            new ResponseListener() {
                @Override
                public void onSuccess(Response response) {
                    
                    latch.countDown();
                }

                @Override
                public void onFailure(Exception exception) {
                    
                    latch.countDown();
                }
            }
    );
}
latch.await();
```

### 读取 Response

```java
Response response = restClient.performRequest(new Request("GET", "/"));
RequestLine requestLine = response.getRequestLine(); 
HttpHost host = response.getHost(); 
int statusCode = response.getStatusLine().getStatusCode(); 
Header[] headers = response.getHeaders(); 
String responseBody = EntityUtils.toString(response.getEntity()); 
```

> getRequestLine()，获取请求的行 (示例：GET app_article_analyse/_count)
>
> getHost()，获取返回 Response 的节点地址
>
> getStatusCode()，获取请求的状态码
>
> getHeaders()，获取响应头
>
> getEntity()，获取响应体

### 配置

**timeout**

```java
RestClientBuilder builder = RestClient.builder(
    new HttpHost("localhost", 9200))
    .setRequestConfigCallback(
        new RestClientBuilder.RequestConfigCallback() {
            @Override
            public RequestConfig.Builder customizeRequestConfig(
                    RequestConfig.Builder requestConfigBuilder) {
                return requestConfigBuilder
                    .setConnectTimeout(5000)
                    .setSocketTimeout(60000);
            }
        })
    .setMaxRetryTimeoutMillis(60000);
```

> 设置连接超时时间为 5s，默认 1s
>
> 设置 socket 超时时间为 60s，默认 30s
>
> 设置重试超时时间为 60s，默认 30s

**线程数**

默认情况下 Apache Async Client 会启动一个调度线程和多个连接管理器使用的工作线程，工作线程的数量跟本地检测到的处理器数量相同(Runtime.getRuntime().availableProcessors() 返回值)。可以通过下面的方法来修改返回的线程数：

```java
RestClientBuilder builder = RestClient.builder(
    new HttpHost("localhost", 9200))
    .setHttpClientConfigCallback(new HttpClientConfigCallback() {
        @Override
        public HttpAsyncClientBuilder customizeHttpClient(
                HttpAsyncClientBuilder httpClientBuilder) {
            return httpClientBuilder.setDefaultIOReactorConfig(
                IOReactorConfig.custom()
                    .setIoThreadCount(1)
                    .build());
        }
    });
```

### Sniffer

自动从正在运行的 Elasticsearch 集群获取节点，并这设置到 `RestClient` 实例上。

引入依赖：

- Maven configuration

  ```xml
  <dependency>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>elasticsearch-rest-client-sniffer</artifactId>
      <version>6.5.4</version>
  </dependency>
  ```

- Gradle configuration

  ```gradle
  dependencies {
      compile 'org.elasticsearch.client:elasticsearch-rest-client-sniffer:6.5.4'
  }
  ```

Sniffer 使用

```java
RestClient restClient = RestClient.builder(
    new HttpHost("localhost", 9200, "http"))
    .build();
Sniffer sniffer = Sniffer.builder(restClient).build();
```

> Sniffer 会周期获取当前的集群节点 (默认 5 分钟)，然后通过 `RestClient.setNode` 方法设置到 `RestClient`

使用完后要关闭 Sniffer:

```java
sniffer.close();
restClient.close();
```

Sniffer 获取节点的周期也可以修改：

```java
RestClient restClient = RestClient.builder(
    new HttpHost("localhost", 9200, "http"))
    .build();
Sniffer sniffer = Sniffer.builder(restClient)
    .setSniffIntervalMillis(60000).build();
```

> 单位是 ms

也可以对节点的失败进行嗅探，会在节点失败的时候马上更新节点列表，而不是在正常的下一个周期中更新：

```java
SniffOnFailureListener sniffOnFailureListener =
    new SniffOnFailureListener();
RestClient restClient = RestClient.builder(
    new HttpHost("localhost", 9200))
    .setFailureListener(sniffOnFailureListener) 
    .build();
Sniffer sniffer = Sniffer.builder(restClient)
    .setSniffAfterFailureDelayMillis(30000) 
    .build();
sniffOnFailureListener.setSniffer(sniffer);
```



