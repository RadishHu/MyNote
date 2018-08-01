# CDH集群安装

## linux环境配置

## 安装CM(Cloudera Manager)

- 安装Apache httpd web服务器(root操作)

  ```shell
  #启动httpd
  service httpd start
  
  #开机启动ghttpd
  chkconfig httpd on
  ```

- 下载CM资源包

  ```
  wget http://archive-primary.cloudera.com/cm5/installer/5.4.1/cloudera-manager-installer.bin
  ```

  