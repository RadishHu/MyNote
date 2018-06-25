# 使用JDK_API发送请求

- 发送GET请求

  ```java
  public class JdkGet {
      public static void main(String[] args) throws IOException {
          //指定URL
          String domain = "http://www.baidu.com/s?wd=北京";
          //发起一个请求，并携带参数?wd=北京
          URL url = new URL(domain);
          HttpURLConnection conn = (HttpURLConnection)url.openConnection();
          //获取一个输入流,字节流
          InputStream inputStream = conn.getInputStream();
          //将字节流转换成字符流
          BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, Charset.forName("utf-8")));
          //打印结果
          String line = null;
          while((line = reader.readLine()) != null) {
              System.out.println(line);
          }
      }
  }
  ```

- 发送POST请求

  ```java
  public class JdkPost {
      public static void main(String[] args) throws IOException {
          //指定URL
          String domain = "http://www.baidu.com/s";
          //发起一个请求
          URL url = new URL(domain);
          HttpURLConnection conn = (HttpURLConnection)url.openConnection();
  
          //设置请求头
          conn.setRequestProperty("User-Agent", "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36");
          //指定请求类型
          conn.setDoOutput(true);
          //设置请求参数
          OutputStream outputStream = conn.getOutputStream();
          outputStream.write("wd=12306".getBytes());
          outputStream.close();
  
          //获取一个字节输入流
          InputStream inputStream = conn.getInputStream();
          BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, Charset.forName("utf-8")));
          String line = null;
          while ((line = reader.readLine()) != null) {
              System.out.println(line);
          }
      }
  }
  ```