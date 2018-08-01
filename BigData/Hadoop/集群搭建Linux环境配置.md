# 集群搭建Linux环境配置

- 关闭防火墙与SELinux

  关闭防火墙：

  ```shell
  service iptables stop
  chkconfig iptables off
  chkconfig iptables --list
  ```

  关闭SELinux：

  ```shell
  #查看SELinux状态
  getenforce
  
  #关闭SELinux
  vim /etc/sysconfig/selinux
  
  #修改
  vim /etc/sysconfig/selinux
  ```

- 网络配置

  ```
  vim /etc/sysconfig/network-scripts/ifcfg-eth0
  ```

  设置静态ip，以及指定ip地址：

  ```
  DEVICE="eth0"
  BOOTPROTO="static"
  IPADDR=192.168.1.110
  NM_CONTROLLED="yes"
  ONBOOT="yes"
  TYPE="Ethernet"
  DNS1=8.8.8.8
  DNS2=8.8.4.4
  GATEWAY=192.168.1.1
  ```

- 修改hosts文件

  ```shell
  vim /etc/hosts
  ```

  把所有要添加到集群中的主机都加入hosts：

  ```shell
  127.0.0.1       localhost
  
  # CDH Cluster
  192.168.1.110   master
  192.168.1.111   slave1
  192.168.1.112   slave2
  ```

- 配置节点之间的ssh

  

  