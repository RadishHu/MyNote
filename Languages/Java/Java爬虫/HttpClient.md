# HttpClient

- pom.xml

  ```xml
  <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
      <version>4.5.4</version>
  </dependency>
  ```

- 发送GET请求

  ```java
  public class HttpClientGet {
      public static void main(String[] args) throws IOException {
          //指定url
          String domain = "http://www.baidu.com/s?wd=北京";
          //打开连接
          CloseableHttpClient httpClient = HttpClients.createDefault();
          HttpGet httpGet = new HttpGet(domain);
          CloseableHttpResponse response = httpClient.execute(httpGet);
          //获取数据
          HttpEntity entity = response.getEntity();
          //打印返回结果
          String html = EntityUtils.toString(entity, Charset.forName("utf-8"));
          System.out.println(html);
      }
  }
  ```

- 发送POST请求

  ```java
  public class HttpClientPost {
      public static void main(String[] args) throws IOException {
          //指定URL
          String domain = "http://www.baidu.com/s";
          //打开连接
          CloseableHttpClient httpClient = HttpClients.createDefault();
          HttpPost httpPost = new HttpPost(domain);
  
          //设置请求头
          httpPost.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36");
          //设置请求参数
          ArrayList<BasicNameValuePair> parameters = new ArrayList<BasicNameValuePair>();
          parameters.add(new BasicNameValuePair("username","hang"));
          parameters.add(new BasicNameValuePair("password","123123"));
          httpPost.setEntity(new UrlEncodedFormEntity(parameters));
  
          CloseableHttpResponse response = httpClient.execute(httpPost);
  
          //获取response数据
          HttpEntity entity = response.getEntity();
          //打印response数据
          String html = EntityUtils.toString(entity, Charset.forName("utf-8"));
          System.out.println(html);
      }
  }
  ```