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

- map逻辑完之后，将map的每条结果通过Context.write进行collect数据收集。在collect中，会对其进行分区处理，默认使用HashPartitioner

  MapReduce提供Partitioner接口，它的作用是根据key或value以及reduce数量来决定当前的这对输出数据最终应该由哪个reduce task处理。默认对key hash以后再以reduce task数量取模。用户可以自定义分区方法并设置在job上

- 然后，会将数据写入内存，内存中这篇区域叫环形缓冲区，缓冲区的作用是批量收集map结果，减少磁盘IO。key-value以及partition的结果都会被写入缓冲区，写入之前key、value的值都会被序列化成字节数组

  > 缓冲区有大小限制，默认100M

  从内存往磁盘写数据的过程成为spill(溢写)，溢写由单独的线程完成，不会影响往缓冲区写map结果的线程。缓冲区有个溢写比例spill.percent，默认0.8，即当缓冲区的数据达到80M，一些线程启动，锁定这80M内存，执行溢写线程，Map task的输出结果可以往剩下20M内存中写

- 溢写线程启动后，会对这80M空间内的key做排序(sort)

  如果job设置了Combiner，会在这里执行Combiner。将有相同key的k-v对的value加起来，减少溢写到数据盘的数据量

- 每次溢写会在磁盘上生成一个临时文件，的那个整个数据处理结束之后，开始对磁盘中临时文件进行merge合并，并进行分区且排序

## Reduce Task

Reduce大致分为copy、sort、reduce三个阶段

- Copy阶段，拉去数据。Reduce进程启动数据copy线程(Fetcher)，通过Http GET请求maptask，获取属于自己的文件

- Merge阶段，copy过来的数据会先放入内存缓冲区，这个缓冲区的大小相比map端的更加灵活。

  > Merge有三种形式：内存到内存(默认不启用)、内存到磁盘、磁盘到磁盘。

  当内存中的数据量到达一定阀值，就启动内存到磁盘的merge。与map端类似，这也是溢写的过程，这个过程如果设置有Combiner也会启动。第二种merge方式一直在运行，知道没有map端的数据时才结束，然后启动第三种磁盘到磁盘的merge方式生成最终文件

- 把分散的数据合并成一个大的数据后，再对合并后数据按照key字典序排序

- 对排序后的k-v对调用reduce方法，键相同的k-v对调用一次reduce方法，最后把输出的键值对写入到HDFS文件中