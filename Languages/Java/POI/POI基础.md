# POI基础

## pom依赖

```xml
<!-- https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.9</version>
</dependency>
```

## 基础操作

- 创建

  创建工作簿

  ```java
  XSSFWorkbook workbook = new XSSFWorkbook();
  ```

  创建工作表

  ```java
  XSSFSheet sheet = workbook.createSheet("sheetName");
  ```

  创建Row

  ```java
  XSSFRow row = sheet.createRow(rowIndex);
  ```

  创建Cell

  ```java
  XSSFCell cell = rowHeader1.createCell(cellIndex);
  ```

  Row和Cell的索引都是从0开始







