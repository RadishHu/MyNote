# 解析Json文件

- 导入pom依赖

  ```xml
  <dependencies>
      <dependency>
          <groupId>com.google.code.gson</groupId>
          <artifactId>gson</artifactId>
          <version>2.6.1</version>
      </dependency>
  </dependencies>
  ```

- 代码实现

  ```java
  //读取文件
  JsonArray array = (JsonArray) parser.parse(new FileReader("filePath"));
  for(int i = 0; i < array.size(); i++) {
      //获取Json数组中的一个元素
      JsonObject object = array.get(i).getAsJsonObject();
      
      String country = object.get("name").getAsJsonObject().get("common").getAsString();
      JsonArray latlng = object.get("latlng").getAsJsonArray();
      String latitude = latlng.get(1).getAsString();
      latlngList.add(latitude);
      String longitude = latlng.get(0).getAsString();
      
  }
  ```

  