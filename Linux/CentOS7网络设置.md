# CentOS7设置静态IP

- 使用背景介绍

  使用虚拟机安装CentOS7，使用NAT网络模式，为了防止再次启动时网络IP发生变化，因此设置静态IP和DNS

  CentOS是最小化安装，没有ifconfig命令，可以使用ip命令查看

  ```
  ip address
  ```

  CentOS刚安装后默认是不启动网路连接，所以只有一个LOOPBACK的127.0.0.1的回环地址

- 修改配置

  修改/etc/sysconfig/network-scripts/目录下以ifcfg开头的，ifcfg-lo是LOOPBACK	网络，修改另一个

  ```
  BOOTPROTO=static #dhcp改为static（修改）
  ONBOOT=yes #开机启用本配置，一般在最后一行（修改）

  IPADDR=192.168.1.204 #静态IP（增加）
  GATEWAY=192.168.1.2 #默认网关，虚拟机安装的话，通常是2，也就是VMnet8的网关设置（增加）
  NETMASK=255.255.255.0 #子网掩码（增加）
  DNS1=192.168.1.2 #DNS 配置，虚拟机安装的话，DNS就网关就行，多个DNS网址的话再增加（增加）
  ```

- 重启网络服务

  ```
  service network restart
  ```

- 安装ifconfig

  ```
  yum search ifconfig
  ```

  发现ifconfig命令是在net-tools.x86_64包中，安装这个包

  ```
  yum -y install net-tools.x86_64
  ```