# CentOS7 yum安装MySQL

## yum安装失败

yum安装mysql：

```
yum -y install mysql-server
```

安装失败：

```
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.sina.cn
 * extras: mirrors.sina.cn
 * updates: mirrors.sina.cn
No package mysql-server available.
Error: Nothing to do
```

失败原因：

CentOS7将MySQL从默认程序列表移除，用mariadb代替

## 安装方法

- 添加mysql到repo：

  ```
  sudo rpm -Uvh https://repo.mysql.com//mysql80-community-release-el7-1.noarch.rpm
  ```

  官方下载地址：https://dev.mysql.com/downloads/repo/yum/

- 安装mysql：

  ```
  yum -y install mysql-server
  ```

- 启动mysql：

  ```
  systemctl start mysqld.service
  ```

## 修改临时密码

- 获取临时密码：

  MySQL会为用户随机生成一个密码，在error log中，使用RPM包安装，error log的默认位置是`/var/log/mysql.log`：

  ```
  grep 'temporary password' /var/log/mysql.log
  
  2018-07-26T05:37:32.238729Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: ot+jnfvt;1CD
  ```

  > 随机密码为：ot+jnfvt;1CD

- 登录并修改密码：

  ```
  mysql -uroot -p
  
  ALTER USER 'root'@'localhost' IDENTIFIED BY 'root123';
  ```

  