# SecondaryNameNode

SecondaryNameNode是hadoop1.x中HDFS HA的一个解决方案

- NameNode始终在内存中存储元数据(metedata)，使读操作更快，有写操作时，向edit文件写入日志，返回成功后修改内存
- NameNode管理着元数据信息，这些信息保存在两个文件中：操作日志文件edits、元数据镜像文件fsimage
  - fsimage为metedata的镜像，不会随时同步，与edits合并后生成新fsimage
  - edits不会随时与fsimage合并生成新的fsimage，因为很消耗资源
- SecondaryNameNode设置了一个checkpoint，当edits文件的大小达到一个临界点(10万条事务操作 或 间隔一段时间，默认1小时)，checkpoint会触发Secondary工作

