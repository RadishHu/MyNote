# Elasticsearch 6.5 安装

- 下载

  ```shell
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.1-linux-x86_64.tar.gz
  ```

- 解压到 /mnt/service 目录下

  ```shell
  tar zxvf elasticsearch-7.0.0-linux-x86_64.tar.gz -C /mnt/service
  ```

- 创建用户和分组

  创建分组

  ```shell
  groupadd elastic
  ```

  添加用户

  ```shell
  useradd -g elastic elasticsearch
  ```

  设置用户密码

  ```shell
  passwd elasticsearch
  ```

- 将 `$ES_HOME` 目录授权给 `elasticsearch` 用户

  ```shell
  chown -R elasticsearch:elastic /mnt/service/elasticsearch-7.1.1-linux-x86_64
  ```

- 修改配置文件 conf/elasticsearch.yml

  ```yml
  # 集群名
  cluster.name: my-es
  # 节点名, 不是服务器的 hostname
  node.name: es-node-01
  # 数据保存目录
  path.data: /var/data/es
  # 日志保存目录
  path.logs: /var/logs/es
  # 禁止使用 swap
  bootstrap.memory_lock: true
  # 本机 ip
  network.host: 192.168.1.10
  # http 暴露端口
  http.port: 9200
  # 配置集群机器
  discovery.zen.ping.unicast.hosts: 
   - 192.168.1.10
   - 192.168.1.11
   - 192.168.1.12
  # 设置为 (master_eligible_node / 2) + 1
  discovery.zen.minimum_master_nodes: 2
  ```

- 修改机器配置

  /etc/security/limits.conf

  ```conf
  # 最大锁定内存地址空间
  elasticsearch soft memlock unlimited
  elasticsearch hard memlock unlimited
  
  # 打开文件的最大数目
  * soft nofile 65536
  * hard nofile 262144
  
  # 进程的最大数目
  * soft nproc 32000
  * hard nproc 32000
  ```

  > 配置中的 elasticsearch 为启动 elasticsearch 时使用的用户名

  设置最大虚拟内存大小 - /etc/sysctl.conf

  ```conf
  vm.max_map_count=655360
  ```

- 启动

  ```shell
  $ su elasticsearch
  $ bin/elasticsearch -d
  ```

  > -d，后台启动