# CentOS7 yum安装MySQL

## 目录

> * [yum安装失败](#chapter1)
> * [安装方法](#chapter2)
> * [修改密码](#chapter3)
> * [远程连接授权](#chpater4)
> * [文件路径](#chapter5)

## yum安装失败 <a id="chapter1"></a>

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

## 安装方法 <a id="chapter2"></a>

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

## 修改临时密码 <a id="chapter3"></a>

- 获取临时密码：

  MySQL会为用户随机生成一个密码，在error log中，使用RPM包安装，error log的默认位置是`/var/log/mysql.log`：

  ```
  grep 'temporary password' /var/log/mysql.log
  ```

  > 2018-07-26T05:37:32.238729Z 5 \[Note]\[MY-010454][Server] A temporary password is generated for root@localhost: ot+jnfvt;1CD
  >
  > 随机密码为：ot+jnfvt;1CD

- 登录并修改密码：

  ```
  mysql -uroot -p
  
  ALTER USER 'root'@'localhost' IDENTIFIED BY 'root123';
  ```

  如果密码太简单会报错：`ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`

  解决方案：

  - 修改validate_password_policy参数的值：

    ```
    mysql> set global validate_password_policy=0;
    ```

  - 修改密码长度：

    ```
    set global validate_password_length=1;
    ```

  - 再次执行修改密码：

    ```
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'root123';
    ```

## 开启数据库远程连接权限 <a id="chapter4"></a>

```sql
use mysql;
grant all privileges on *.* to root@"%" identified by "password" with grant option;
flush privileges;
```

> `%`代表允许所有的远程连接
>
> `by "root"`代表数据库的密码

MySQL8.0授权方式有所不同，参考：[MySQL8.0授权方式](/Database/MySQL/MySQL8.0.md)

## 文件路径 <a id="chapter5"></a>

CentOS7下安装mysql的默认路径：

- 配置文件：/etc/my.cof
- 日志文件：/var/log/mysqld.log
- 服务启动脚本：/usr/lib/systemd/system/mysqld.service
- pid文件：/var/run/mysqld/mysqld.pid
- socket文件：/var/lib/mysql/mysql.sock
- 数据文件：/var/lib/mysql