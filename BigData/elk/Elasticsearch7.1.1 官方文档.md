# 配置 Elasticsearch

## 重要的配置

### path

- path.logs
- path.data

如果是通过 **.zip** 或 **.tar.gz** 进行安装，**data** 和 **logs** 目录默认是在 **$ES_HOME** 目录下，如果把这些目录目录放在默认的位置，在升级 Elasticsearch 到新版本时，很有可能会被删除。在生产环境中，最好对这两个目录的位置进行修改：

```yml
path:
 logs: /var/log/elasticsearch
 data: /var/data/elasticsearch
```

> 如果通过 **RPM** 进行的安装，已经使用这个位置来保存这个两个目录了

`path.data` 可以设置为多个目录，这样所有的目录都会保存数据，属于同一个分片的文件会被保存在相同的数据目录下：

```yml
path:
 data:
  - /mnt/elasticsearch_1
  - /mnt/elasticsearch_2
  - /mnt/elasticsearch_3
```

### cluster.name

默认值为 **elasticsearch**

```yml
cluster.name: logging-prod
```

> 不要在不同的集群中使用相同的名字，这样节点会加入到错误的集群中

### node.name

集群中节点的名字

```yml
node.name: pro-data-2
```

### network.host

可以设置为节点的 ip 或 hostname

```yml
network.host: 192.168.1.10
```

### discovery

- discovery.seed_hosts
- cluster.initial_master_nodes



