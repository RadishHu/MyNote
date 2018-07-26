# cmake安装

- 下载安装包

  官网下载：[https://cmake.org/download/](https://link.jianshu.com/?t=https://cmake.org/download/) 

- 解压安装包

  ```shell
  tar zxvf cmake-3.11.4.tar.gz
  ```

- 进入文件目录，执行`./bootstrap`

  遇到问题：

  ```
  ---------------------------------------------
  CMake 3.10.0, Copyright 2000-2017 Kitware, Inc. and Contributors
  C compiler on this system is: cc  
  ---------------------------------------------
  Error when bootstrapping CMake:
  Cannot find a C++ compiler supporting C++11 on this system.
  Please specify one using environment variable CXX.
  See cmake_bootstrap.log for compilers attempted.
  ---------------------------------------------
  Log of errors: /home/javascript/下载/cmake-3.10.0/Bootstrap.cmk/cmake_bootstrap.log
  ---------------------------------------------
  ```

  问题原因：没有安装c++编译器，解决方案：

  ```
  Ubuntu：apt-get install gcc g++
  CentOS：yum install -y gcc gcc-c++
  ```

  再次执行`./bootstrap`

- 执行`make`，进行编译

- 执行`make install`

- 查看版本

  ```
  cmake --version
  ```