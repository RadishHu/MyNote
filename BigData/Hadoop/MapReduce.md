# MapReduce

MapReduce程序在分布式运行时会有三类示例进程：

- MRAppMaster：负责整个程序的过程调度及状态协调
- MapTask：负责Map阶段的整个数据处理流程
- ReduceTask：负责Reduce阶段的整个数据处理流程

## MapTask

Map阶段执行过程：

- 读取数据组件InputFormat通过getSplits方法对输入目录中文件进行逻辑切片，规划得到splits有多少个split，就对应启动多少个MapTask，split切片大小默认为block块大小
- 将输入文件切分为splits后，由RecordReader对象进行读取，以`\n`作为分隔符，读取一行数据，返回k-v对，key为每行首字符的偏移值，value为这一行文本内容
- 读取split返回的k-v对，进入用户自己继承的Mapper类中，执行用户重写的map函数，RecordRead读取一行，这里调用一次
- map逻辑完之后，将map的每条结果通过Context.write进行collect

