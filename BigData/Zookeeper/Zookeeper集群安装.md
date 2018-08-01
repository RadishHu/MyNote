# Zookeeper集群安装

- 下载、解压

  ```shell
  wget http://124.205.69.163/files/1171000003E0B205/archive.apache.org/dist/zookeeper/zookeeper-3.4.5/zookeeper-3.4.5.tar.gz
  tar -zxvf zookeeper-3.4.5.tar.gz -C /export/servers
  ```

- 修改环境变量

  ```
  vi /etc/profile
  
  #添加内容
  export ZOOKEEPER_HOME=/export/servers/zookeeper
  export PATH=$PATH:$ZOOKEEPER_HOME/bin
  
  #重新编译
  source /etc/profile
  ```

- 修改zookeeper配置文件

  修改zoo.cfg：

  ```
  cd /export/servers/zookeeper/conf
  mv zoo_sample.cfg zoo.cfg
  vi zoo.cfg
  
  #添加内容
  dataDir=/export/servers/zookeeper/data
  dataLogDir=/export/servers/zookeeper/log
  server.1=slave1:2888:3888 
  server.2=slave2:2888:3888
  server.3=slave3:2888:3888
  ```

  创建文件夹：

  ```
  mkdir /export/servers/zookeeper/data
  mkdir /export/servers/zookeeper/log
  ```

  创建myid文件，并添加内容：

  ```
  cd /export/servers/zookeeper/data
  vi myid
  
  #添加内容
  1
  ```

  修改hosts文件

  ```
  vim /etc/hosts
  ```

- 运行zookeeper

  ```
  /export/servers/zookeeper/bin/zkServer.sh start
  ```

- 产看运行状态

  ```
  jps
  zkServer.sh status
  ```

- 使用Observer模式

  在对应节点的配置文件添加：

  ```
  peerType=observer
  ```

  在配置文件指定那些节点为Observer：

  ```
  server.1:localhost:2181:3181:observer
  ```

  