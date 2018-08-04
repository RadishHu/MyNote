# Znode

znode存储数据大小有限制，zk的服务器和客户端都被设计为严格检查并限制每个znode的数据大小之多1M

## 目录

> * [znode组成](#chapter1)
> * [znode类型](#chapter2)

## znode的组成 <a id="chapter1"></a>

每个znode由3部分组成：

- stat：状态信息，描述该znode的版本、权限等信息
- data：与该znode关联的数据
- children：该znode下的子节点

## znode类型 <a id="chapter2"></a>

znode可以分为临时节点和永久节点，节点类型在创建是即被确定，且不能改变：

- 临时节点

  该节点的生命周期依赖于创建它们的会话，一旦会话结束，临时节点将被自动删除，也可以手动删除，临时节点不允许拥有子节点

- 永久节点

  该节点的生命周期不依赖于会话，并且只有在客户端显示执行删除时，才会被删除

znode有一个序列化的特性，创建时指定的话，znode的名字后面会自动追加一个不断增加的序列号，序列号对于此父节点来说是唯一的，这样便会记录每个子节点创建的先后顺序。这样就会有四种类型的znode节点：

- PERSISTENT：永久节点
- EPHEMERAL：临时节点
- PERSISTENT_SEQUENTIAL：永久节点_序列化
- EPHEMERAL_SEQUENTIAL：临时节点_序列化