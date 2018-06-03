# CentOS Docker安装

- CentOS版本要求

  CentOS7,要求系统为64位，系统内核为3.10以上

  CentOS6.5或更高版本，要求系统64位，系统内核版本为2.6.32-432或更高版本

  查看内核版本：

  ```shell
  uname -r
  ```

- 安装Docker

  ```shell
  yum -y install docker-io
  ```

- 启动Docker后台服务

  ```
  service docker start
  ```

- 镜像加速

  使用网易镜像：http://hub-mirror.c.163.com

  修改配置文件/etc/docker/daemon.json

  ```
  {
    "registry-mirrors": ["http://hub-mirror.c.163.com"]
  }
  ```

