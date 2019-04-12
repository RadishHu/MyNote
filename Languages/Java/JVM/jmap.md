# jmap



jmap 命令可以生成 java 程序的 dump 文件、查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列



## jmap 语法 <a id="chapter1"></a>

jmap 命令语法：

```
jmap [option] <pid>
```

> option:
>
> - no option: 打印进程的内存映射信息，类似 `Solaris pmap` 命令
>
> - -heap: 打印 java 堆的配置情况、使用情况以及使用的 GC 算法
>
> - -histo[:live]: 打印 java 堆中各个对象的数量、大小；如果加上 `:live` ，只统计活跃的对象
>
> - -clstats: 打印类加载器的信息
>
> - -finalizerinfo: 打印正在等待执行 finalize 方法的对象
>
> - -dump:\<dump-options>: 把 java 堆中的对象 dump 到本地文件
>
>   dump-options:
>
>   - live: 只 dump 活跃的对象
>   - format=b: 以二进制格式保存
>   - file=\<file>: 指定 dump 到本地的文件
>
>   示例：
>
>   ```
>   jmap -dump:live,format=b,file=heap.bin <pid>
>   ```
>
>   
>
> - -J\<flag>: 传递参数给正在运行的 JVM

