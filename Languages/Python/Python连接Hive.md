# Python 连接 Hive

这里通过 PyHive 来连接 Hive，在连接 Hive 之前需要先安装几个包：

```python
pip install sasl
pip install thrift
pip install thrift-sasl

pip install PyHive
```

> - Windows 安装 sasl 时可能会报错，需要下载 [sasl](https://www.lfd.uci.edu/~gohlke/pythonlibs/#sasl)，选择对应的版本，`cmd` 进入下载的目录
>
>   ```
>   pip install sasl-0.2.1-cp27-cp27m-win_amd64
>   ```
>
> - CentOS7 安装 sasl 时也可能会报错：
>
>   - sasl/saslwrapper.cpp:8:22: fatal error: pyconfig.h: No such file or directory
>
>     ```
>     yum install python-devel.x86_64
>     ```
>
>   - sasl/saslwrapper.h:22:23: fatal error: sasl/sasl.h: No such file or directory
>
>     ```
>     yum install gcc-c++ python-devel.x86_64 cyrus-sasl-devel.x86_64
>     ```





