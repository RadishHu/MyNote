# HDFS工作机制 

## 目录

> * [HDFS写数据流程](#chapter1)
> * [HDFS读数据流程](#chapter2)

HDFS工作机制：

- NameNode：负责管理整个文件系统元数据
- DataNode：负责管理具体文件数据块存储
- SecondaryNameNode：协助NameNode进行元数据备份

NameNode：

- NameNode并不会持久化存储每个文件中各个块所在的DataNode的位置信息，这些信息会在信息会在系统启动时从数据节点重建
- NameNode是Hadoop集群找中的单点故障
- NameNode所在的机器通常会配置有大量内存

DataNode：

- DataNode启动时，会将自己发布到NameNode并汇报自己持有的块列表，block汇报时间间隔参数`dfs.blockreprot.intervalMsec`，默认6小时
- DataNode所在机器通常配置有大量的硬盘空间
- DataNode会定期(`dfs.heartbeat.interval`，默认3秒)向NameNode发送心跳，如果NameNode长时间没有接收到DataNode发送的心跳，NameNode就会认为该DataNode失效

## HDFS写数据流程 <a id="chapter1"></a>

- Client发起文件上传请求，通过RPC与NameNode建立通讯，NameNode检查目标文件是否已存在，父目录是否已存在，返回是否可以上传

- client请求第一个block传输到哪些DataNode服务器上

- NameNode根据配置文件指定备份数量及机架感知原理进行文件分配，返回可用的DataNode的地址，如：A、B、C

  > Hadoop在设计时考虑到数据的安全与高效，数据默认在HDFS上存放三份，存储策略为本地一份，同机架内其他某一节点上一份，不同机架上一份

- client请求3台DataNode中一台A上传数据(本质上时RPC调用，建立pipeline)，A收到请求会继续调用B，然后B调用C，将整个pipeline建立完成，后逐级返回client

- client开始往A上传第一个block，数据传输以packet为单位(先从磁盘读取数据存放到本地一个内存缓存，默认大小64K)，A收到一个packet就会传给B，B传给C，A每传一个packet会放入一个应答队列等待应答

- 数据分割成一个个packet数据包在pipeline上一次传输，在pipeline反方向上，逐个发送ack，最终由pipeline中第一个DataNode节点A将pipeline ack发送给client

- 当一个block传输完成后，client再次请求NameNode上传第二个block

- 所有的数据都写完后，通知NameNode，将这条信息同步到文件系统元数据中

## HDFS读数据流程 <a id="chapter2"></a>

- client向NameNode发起RPC请求，来确定文件block所在的位置

- NameNode返回文件block列表(如果block块文件较多，会先返回部分列表)，对于每个block，NameNode都会返回含有该block副本的所有DataNode地址

  > 返回的DataNode地址会进行排序，排序规则：网络拓扑结构中距离client近的排靠前，心跳机制中超时汇报的DataNode状态为STALE,排序靠后

- client选取排序靠前的DataNode来读取block，底层本质是建立Socket Stream，重复调用父类DataInputStream的read方法，直到这个block上的数据读取完毕

- 读完列表中的block后，若文件读取还没有结束，客户端会继续向NameNode获取下一批的block列表

- 读取完一个block，会进行checksum验证，如果读取DataNode是出现错误，客户端会通知NameNode，然后再从下一个有该block副本的额DataNode继续读

- read方法是并行的读取block方法，不是一块一块的读取

- 最终读取来的所有block会合并(merge)成一个完整的最终文件