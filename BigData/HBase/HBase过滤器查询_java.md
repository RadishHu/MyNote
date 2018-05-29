# HBase过滤器查询—Java

HBase过滤器可以分为两类：比较过滤器，专用过滤器

- HBase过滤器的比较运算符：

  ```
  LESS				<
  LESS_OR_EQUAL		<=
  EQUAL				=
  NOT_EQUAL			<>
  GREATER_OR_EQUAL	>=
  GREATER				>
  NOT_OP				排除所有
  ```

- HBase过滤器的比较器，用于指定比较机制

  ```
  BinaryComparator 按字节索引顺序比较指定字节数组，采用Byte.compare(byte[])
  BinaryPrefixComparator 跟上面的相同，只是比较左端的数据是否相同
  NullComparator 判断给定的是否为空
  BitComparator  按位比较
  RegexStringComparator 提供一个正则比较器，仅支持EQUAL和非EQUAL
  SubstringComparator 判断提供的子串是否出现在value中
  ```

## 比较过滤器

- 行键过滤器 RowFilter

  ```java
  Filter filter = new RowFilter(CompareFilter.CompareOp.LESS_OR_EQUAL, new BinaryComparator(Bytes.toBytes("rowkey")));  
  scan.setFilter(filter);
  ```

- 列族过滤器 FamilyFilter

  ```java
  Filter filter = new FamilyFilter(CompareFilter.CompareOp.LESS, new BinaryComparator(Bytes.toBytes("family")));
  scan.setFilter(filter1); 
  ```

- 列过滤器 QualifierFilter

  ```java
  Filter filter = new QualifierFilter(CompareFilter.CompareOp.LESS_OR_EQUAL, new BinaryComparator(Bytes.toBytes("column")));
  scan.setFilter(filter);
  ```

- 值过滤器 ValueFilter

  ```java
  Filter filter = new ValueFilter(CompareFilter.CompareOp.EQUAL, new SubstringComparator(".4") );  
  scan.setFilter(filter);
  ```

## 专用过滤器

- 单值过滤器 SingleColumnValueFilter，返回满足条件的整行

  ```java
  SingleColumnValueFilter filter = new SingleColumnValueFilter(  
      Bytes.toBytes("family"),  
      Bytes.toBytes("column"),  
      CompareFilter.CompareOp.NOT_EQUAL,  
      new SubstringComparator("val-5"));  
  filter.setFilterIfMissing(true);  //如果不设置为true，则那些不包含指定column的行也会返回
  scan.setFilter(filter); 
  ```

- SingleColumnValueExcludeFilter，与上相反

- 前缀过滤器 PrefixFilter，针对rowkey

  ```java
  Filter filter = new PrefixFilter(Bytes.toBytes("rowkey"));  
  scan.setFilter(filter)
  ```

## 过滤器使用示例

```java
@Test
public void testScan() throws Exception{
    HTable table = new HTable(conf,"table".getByte());
    Scan scan = new Scan(Bytes.toBytes("rowkey"),Bytes.toBytes("rowkey"));
    
    //前缀过滤器，针对行键
    Filter filter = new PrefixFilter(Bytes.toBytes("rk"));
    
    //行过滤器，针对行键
    ByteArrayComparable rowComparator = new BinaryComparator(Bytes.toBytes("rowkey"));
    RowFilter rf = new RowFilter(CompareOp.LESS_OR_EQUAL,rowComparator);
    
    /**
    * 假设rowkey格式为：创建日期_发布日期_ID_TITLE
    * 目标：查找  发布日期  为  2014-12-21  的数据
    */
    rf = new RowFilter(CompareOp.EQUAL,new SubstringComparator("_2014-12-21_"));
    
    //单值过滤器，完整匹配字节数组
    new SingleColumnValueFilter(
        "family".getBytes(),
        "column".getBytes(),CompareOp.EQUAL,
        "value".getBytes());
    //单值过滤器，匹配正则表达式
    ByteArrayComparable comparator = new RegexStringComparator("zhang.");
    new SingleColumnValueFilter(
        "family".getBytes(),
        "column".getBytes(),
        CompareOp.EQUAL,comparator);
    //单值过滤器，匹配是否包含子串，大小写不敏感
    comparator = new SubstringComparator("wu");
    new SingleColumnValueFilter(
        "family".getBytes(),
        "column".getBytes(),
        CompareOp.EQUAL,comparator);
    
    //键值对元数据过滤，family过滤，字节数组完整匹配
    FamilyFilter ff = new FamilyFilter(
        CompareOp.EQUAL,
        new BinaryComparator(Bytes.toBytes("family")));//表中不存在该列族，过滤结果为空
    ff = new FamilyFilter(
        CompareOp.EQUAL , 
        new BinaryPrefixComparator(Bytes.toBytes("inf")));//表中存在以inf打头的列族，过滤结果为该列族所有行
    
    //键值对元数据过滤-----qualifier过滤----字节数组完整匹配
    filter = new QualifierFilter(
        CompareOp.EQUAL,
        new BinaryComparator(Bytes.toBytes("na")));//表中不存在na列，过滤结果为空
	filter = new QualifierFilter(
        CompareOp.EQUAL , 
        new BinaryPrefixComparator(Bytes.toBytes("na"))); //表中存在以na打头的列name，过滤结果为所有行的该列数据
	
    //基于列名(即Qualifier)前缀过滤数据
    filter = new ColumnPrefixFilter("na".getBytes());
	
    //基于列名(即Qualifier)多个前缀过滤数据的MultipleColumnPrefixFilter
    byte[][] prefixes = new byte[][] {Bytes.toBytes("na"), Bytes.toBytes("me")};
    filter = new MultipleColumnPrefixFilter(prefixes);

    //为查询设置过滤条件
    scan.setFilter(filter);
    scan.addFamily(Bytes.toBytes("base_info"));

    //获取一行数据
    Result result = table.get(get);
    //获取多行数据
    ResultScanner scanner = table.getScanner(scan);
    for(Result r : scanner){
        for(KeyValue kv : r.list()){
            String family = new String(kv.getFamily());
            System.out.println(family);
            String qualifier = new String(kv.getQualifier());
            System.out.println(qualifier);
            System.out.println(new String(kv.getValue()));
        }
        //直接从result中取到某个特定的value
        byte[] value = r.getValue(Bytes.toBytes("family"), Bytes.toBytes("column"));
        System.out.println(new String(value));
    }
    table.close();
}
```

