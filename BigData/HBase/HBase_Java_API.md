# HBase_Java_API

## pom依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>0.96.2-hadoop2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>2.2.0</version>
    </dependency>
</dependencies>
```

## 基本增删改查

```java
public class HbaseDemo{
    private Configuration conf = null;
    
    @Before
    public void init() {
        conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum","node01:2181,node02:2181,node03:2181")
    }
    
    //删除表
    @Test
    public void testDrop() throws Exception{
        HBaseAdmin admin = new HBaseAdmin(conf);
        admin.disableTable("account");
        admin.deleteTable("account");
        admin.close();
    }
    
    //添加数据
    @Test
    public void testPut() throws Exception{
        HTable table = new HTable(conf,"t1");
        Put p = new Put(Bytes.toBytes("rowkey"));
        p.add("family".getBytes(),"column".getBytes(),"value".getBytes());
        table.put(p);
        table.close();
    }
    
    //删除数据
    @Test
    public void testDel() throws Exception{
        HTable table = new HTable(conf,"t1");
        Delete del = new Delete(Bytes.toBytes("rowkey"));
        del.deleteColumn(Byte.toBytes("family"),Bytes.toBytes("pic"));
        table.delete(del);
        table.close();
    }
    
    //获取数据
    @Test
    public void testGet() throws Exception{
        HTable table = new HTable(conf,"t1");
        Get get = new Get(Bytes.toBytes("rowkey"));
        get.setMaxVersion(5);
        Result result = table.get(get);
        
        List<Cell> cells = result.listCells();
        for(Cell cell : cells) {
            String family = new String(CellUtil.cloneFamily(cell));
            String qualifier = new String(CellUtil.cloneQualifier(cell));
            String value = new String(CellUtil.cloneValue(cell));
        }
        table.close();
    }
}
```

