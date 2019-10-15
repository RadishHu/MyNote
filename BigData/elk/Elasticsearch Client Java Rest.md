# Rest Client

# 概述

Java Rest Client 有两种：

- Low-level Client：通过 **http** 跟 Elasticsearch 连接，它跟所有的 Elasticsearch 版本兼容。
- Hight-level Client：基于低级 client，暴露一些方法

# Low-level Client

## Maven Repository

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

## Shading

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

## 初始化

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

## 发送请求

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

## 读取 Response

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

## 配置

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

## Sniffer

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

Sniffer 在返回 Elasticsearch 集群节点信息时，只会返回 `host:port` 对，不会返回连接的协议，默认使用 `http` 协议。可以通过下面的方法来使用 `https` 协议：

```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
NodesSniffer nodesSniffer = new ElasticsearchNodesSniffer(
        restClient,
        ElasticsearchNodesSniffer.DEFAULT_SNIFF_REQUEST_TIMEOUT,
        ElasticsearchNodesSniffer.Scheme.HTTPS);
Sniffer sniffer = Sniffer.builder(restClient)
        .setNodesSniffer(nodesSniffer).build();
```

# Low-level Client

高级 Rest Client 是对低级 Client 进行封装

## Maven Repository

- Maven configuration

  ```xml
  <dependency>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>elasticsearch-rest-high-level-client</artifactId>
      <version>6.5.4</version>
  </dependency>
  ```

- Gradle configuration

  ```gradle
  dependencies {
      compile 'org.elasticsearch.client:elasticsearch-rest-high-level-client:6.5.4'
  }
  ```

## 初始化

创建 `RestHighLevelClient` 实例：

```java
RestHighLevelClient client = new RestHighLevelClient(
        RestClient.builder(
                new HttpHost("localhost", 9200, "http"),
                new HttpHost("localhost", 9201, "http")));
```

High-level client 内部会创建 low-level client 来执行请求，low-level client 会维护一个连接池和多个线程，因此使用完后需要关闭它：

```java
client.close()
```

## RequestOptions

`RestHigthLevelClient` 可以接收一个 `RequestOptions` 来定制请求参数

## Document API

### Index API

**IndexRequest**

```java
IndexRequest request = new IndexRequest(
        "posts", 
        "doc",  
        "1");   
String jsonString = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
        "}";
request.source(jsonString, XContentType.JSON);
```

> 参数说明：
>
> "posts"，Index 的名字
>
> "Type"，类型
>
> "1"，Document id
>
> jsonString，以 `String` 格式来提供 Document 的数据

**Document source**

可以以多种形式来提供 Document 的数据，上面展示了 `String` 格式。

 `Map` 格式的数据：

```java
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("user", "kimchy");
jsonMap.put("postDate", new Date());
jsonMap.put("message", "trying out Elasticsearch");
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1")
        .source(jsonMap); 
```

> `Map` 格式的数据最终会自动转换为 `JSON` 格式

`XContentBuilder` 格式的数据：

```java
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.field("user", "kimchy");
    builder.timeField("postDate", new Date());
    builder.field("message", "trying out Elasticsearch");
}
builder.endObject();
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1")
        .source(builder);
```

`K-V` 对象：

```java
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1")
        .source("user", "kimchy",
                "postDate", new Date(),
                "message", "trying out Elasticsearch");
```

**可选的参数**

- Routing

  ```java
  request.routing("routing");
  ```

- Parnet

  ```java
  request.parent("parent");
  ```

- Timeout

  ```java
  request.timeout(TimeValue.timeValueSeconds(1));
  request.timeout("1s")
  ```

  > 等待主分片变为可用的超时时间

- Refresh policy

  ```java
  request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL); 
  request.setRefreshPolicy("wait_for"); 
  ```

- Version

  ```java
  request.versionType(VersionType.EXTERNAL);
  ```

- Version type

  ```java
  request.opType(DocWriteRequest.OpType.CREATE); 
  request.opType("create"); 
  ```

- pipeline

  ```java
  request.setPipeline("pipeline")
  ```

**发送请求**

- 发送同步请求

  ```java
  IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);
  ```

- 发送异步请求

  ```java
  client.indexAsync(request, RequestOptions.DEFAULT, new ActionListener<IndexResponse>() {
  
              @Override
              public void onResponse(IndexResponse indexResponse) {
                  System.out.println("请求成功");
              }
  
              @Override
              public void onFailure(Exception e) {
                  System.out.println("请求发送异常");
              }
          });
  ```

**Index Response**

可以从 `IndexResponse` 中获取以下信息：

```java
String index = indexResponse.getIndex();
String type = indexResponse.getType();
String id = indexResponse.getId();
long version = indexResponse.getVersion();
if (indexResponse.getResult() == DocWriteResponse.Result.CREATED) {
    // 1. document 是第一次被创建
} else if (indexResponse.getResult() == DocWriteResponse.Result.UPDATED) {
    // 2. document 被更新
}
ReplicationResponse.ShardInfo shardInfo = indexResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    // 3. 成功的分片数小于总分片数
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure :
            shardInfo.getFailures()) {
        String reason = failure.reason(); 
        // 4. 处理潜在的失败
    }
}
```

如果存在版本冲突，会抛出一个 `ElasticsearchException` 错：

```java
IndexRequest request = new IndexRequest("posts", "doc", "1")
        .source("field", "value")
        .version(1);
try {
    IndexResponse response = client.index(request, RequestOptions.DEFAULT);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        // 存在版本冲突
    }
}
```

> 在发送请求时指定了 version，但是 document 的版本号不一样

如果使用的 `create` 操作，一个 index、type 和 id 都相同的 document 已经存在，也会报 `ElasticsearchException` 错：

```java
IndexRequest request = new IndexRequest("posts", "doc", "1")
        .source("field", "value")
        .opType(DocWriteRequest.OpType.CREATE);
try {
    IndexResponse response = client.index(request, RequestOptions.DEFAULT);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        // 存下相同的 document
    }
}
```

### Get API

**Get Request**

```java
GetRequest getRequest = new GetRequest(
        "posts", 
        "doc",  
        "1");
```

> "posts"，index 名字
>
> "doc"，document 类型
>
> "1"，document id

**可选参数**

- 禁用 source 检索，默认是开启的

  ```java
  request.fetchSourceContext(FetchSourceContext.DO_NOT_FETCH_SOURCE);
  ```

- 配置包含/不包含特定字段的 source

  ```java
  String[] includes = new String[]{"message", "*Date"};
  String[] excludes = Strings.EMPTY_ARRAY;
  FetchSourceContext fetchSourceContext =
          new FetchSourceContext(true, includes, excludes);
  request.fetchSourceContext(fetchSourceContext);
  ```

- 配置特定存储字段的检索

  ```java
  request.storedFields("message"); 
  GetResponse getResponse = client.get(request, RequestOptions.DEFAULT);
  String message = getResponse.getField("message").getValue(); 
  ```

- Routing

  ```java
  request.routing("routing");
  ```

- Parent

  ```java
  request.parent("parent");
  ```

- Preference

  ```java
  request.preference("preference");
  ```

- realtime

  ```java
  request.realtime(false);
  ```

- 设置在检索 document 之前刷新

  ```java
  request.refresh(true);
  ```

  > 默认为 false

- Version

  ```java
  request.version(2);
  ```

- Version type

  ```java
  request.versionType(VersionType.EXTERNAL);
  ```

**发送请求**

- 发送同步请求

  ```java
  GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
  ```

- 发送异步请求

  ```java
  GetResponse response = client.get(getRequest, RequestOptions.DEFAULT);
          client.getAsync(getRequest, RequestOptions.DEFAULT, new ActionListener<GetResponse>() {
              @Override
              public void onResponse(GetResponse documentFields) {
                  
              }
  
              @Override
              public void onFailure(Exception e) {
  
              }
          });
  ```

**Get Response**

```java
String index = getResponse.getIndex();
String type = getResponse.getType();
String id = getResponse.getId();
if (getResponse.isExists()) {
    long version = getResponse.getVersion();
    String sourceAsString = getResponse.getSourceAsString();        
    Map<String, Object> sourceAsMap = getResponse.getSourceAsMap(); 
    byte[] sourceAsBytes = getResponse.getSourceAsBytes();          
} else {
    // 没有查找到 Document
}
```

当要查找的 index 不存在时会抛出 `ElasticsearchException` 错：

```java
GetRequest request = new GetRequest("does_not_exist", "doc", "1");
try {
    GetResponse getResponse = client.get(request, RequestOptions.DEFAULT);
} catch (ElasticsearchException e) {
    if (e.status() == RestStatus.NOT_FOUND) {
        // index 不存在
    }
}
```

当请求指定的 document 版本与真实的版本号不一致时，也会报错：

```java
try {
    GetRequest request = new GetRequest("posts", "doc", "1").version(2);
    GetResponse getResponse = client.get(request, RequestOptions.DEFAULT);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.CONFLICT) {
        // 版本号不一致
    }
}
```

### Exists API

如果 document 存在则返回 **true**，如果不存在则返回 **false**。

**Exists Request**

`exists()` 使用的是 `GetRequest`，`GetRequest` 的所有可算参数都可以使用在这里：

```java
GetRequest getRequest = new GetRequest(
    "posts", 
    "doc",   
    "1");    
getRequest.fetchSourceContext(new FetchSourceContext(false)); 
getRequest.storedFields("_none_");  
```

> 因为 `exists()` 返回的 **true** 或 **false**，所以推荐关闭 **_source** 和 stored fields

**发送请求**

- 发送同步请求

  ```java
  boolean exists = client.exists(getRequest, RequestOptions.DEFAULT);
  ```

- 发送异步请求

  ```java
  client.existsAsync(getRequest, RequestOptions.DEFAULT, new ActionListener<Boolean>() {
              @Override
              public void onResponse(Boolean aBoolean) {
                  
              }
  
              @Override
              public void onFailure(Exception e) {
  
              }
          });
  ```

### Delete API

**Delete Request**

```java
DeleteRequest request = new DeleteRequest(
        "posts",    
        "doc",      
        "1");
```

**可算参数**

- Routing

  ```java
  request.routing("routing");
  ```

- Parent

  ```java
  request.parent("parent");
  ```

- Timeout

  ```java
  request.timeout(TimeValue.timeValueMinutes(2)); 
  request.timeout("2m");
  ```

- 刷新策略

  ```java
  request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL); 
  request.setRefreshPolicy("wait_for");
  ```

- Version

  ```java
  request.version(2);
  ```

- Version Type

  ```java
  request.versionType(VersionType.EXTERNAL);
  ```

**发送请求**

- 发送同步请求

  ```java
  DeleteResponse deleteResponse = client.delete(
          request, RequestOptions.DEFAULT);
  ```

- 发送异步请求

  ```java
  client.deleteAsync(request, RequestOptions.DEFAULT, new ActionListener<DeleteResponse>() {
              @Override
              public void onResponse(DeleteResponse deleteResponse) {
                  
              }
  
              @Override
              public void onFailure(Exception e) {
  
              }
          });
  ```

**Delete Response**

```java
String index = deleteResponse.getIndex();
String type = deleteResponse.getType();
String id = deleteResponse.getId();
long version = deleteResponse.getVersion();
ReplicationResponse.ShardInfo shardInfo = deleteResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    // 处理成功分片数小于总分片数的情况
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure :
            shardInfo.getFailures()) {
        String reason = failure.reason(); 
        // 处理潜在的失败情况
    }
}
```

处理要删除的 document 不存在的情况：

```java
if (deleteResponse.getResult() == DocWriteResponse.Result.NOT_FOUND) {
    // 处理要删除的 document 不存在的情况
}
```

请求的 document 版本跟正式版本冲突时，会抛出 `ElasticsearchException` 错：

```java
try {
    DeleteResponse deleteResponse = client.delete(
            new DeleteRequest("posts", "doc", "1").version(2),
            RequestOptions.DEFAULT);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.CONFLICT) {
        // 版本冲突
    }
}
```

### Update API

**Update Request**

```java
UpdateRequest request = new UpdateRequest(
        "posts", 
        "doc",  
        "1");
```

Update API 可以通过 script 或一个局部的 document 来更新一个已经存在 document

**通过 Script 更新**

- inline script

  ```java
  Map<String, Object> parameters = singletonMap("count", 4); 
  
  Script inline = new Script(ScriptType.INLINE, "painless",
          "ctx._source.field += params.count", parameters);  
  request.script(inline);
  ```

- stored script

  ```java
  Script stored = new Script(
          ScriptType.STORED, null, "increment-field", parameters);  
  request.script(stored);
  ```

**通过局部的 document 更新**

- JSON 格式的字符串

  ```java
  UpdateRequest request = new UpdateRequest("posts", "doc", "1");
  String jsonString = "{" +
          "\"updated\":\"2017-01-01\"," +
          "\"reason\":\"daily update\"" +
          "}";
  request.doc(jsonString, XContentType.JSON);
  ```

- JSON 格式的 Map

  ```java
  Map<String, Object> jsonMap = new HashMap<>();
  jsonMap.put("updated", new Date());
  jsonMap.put("reason", "daily update");
  UpdateRequest request = new UpdateRequest("posts", "doc", "1")
          .doc(jsonMap);
  ```

- `XContentBuilder` 对象

  ```java
  XContentBuilder builder = XContentFactory.jsonBuilder();
  builder.startObject();
  {
      builder.timeField("updated", new Date());
      builder.field("reason", "daily update");
  }
  builder.endObject();
  UpdateRequest request = new UpdateRequest("posts", "doc", "1")
          .doc(builder);
  ```

- K-V 对象

  ```java
  UpdateRequest request = new UpdateRequest("posts", "doc", "1")
          .doc("updated", new Date(),
               "reason", "daily update");
  ```

**Upserts**

如果要更新的 document 不存在，那需要把这个 document 插入，可以使用 `upsert` 方法：

```java
String jsonString = "{\"created\":\"2017-01-01\"}";
request.upsert(jsonString, XContentType.JSON);
```

> 要更新的 document 的内容可以使用  String, Map, XContentBuilder, K-V 对象

**可选参数**

- Routing

  ```java
  request.routing("routing");
  ```

- Parent

  ```
  request.parent("parent");
  ```

- Timeout

  ```java
  request.timeout(TimeValue.timeValueSeconds(1)); 
  request.timeout("1s");
  ```

- 刷新策略

  ```java
  request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL); 
  request.setRefreshPolicy("wait_for");
  ```

  









