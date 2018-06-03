# iptables

## 安装iptables

- 安装iptable iptable-service

  检查是否安装iptables

  ```
  service iptables status
  ```

  安装iptables

  ```
  yum -y install iptables
  ```
  升级iptables(安装的是最新版本不需要)

  ```
  yum update iptable
  ```

  安装iptables-service

  ```
  yum install iptables-serivces
  ```

- 停止/禁用自带的firewalld服务

  停止firewalld服务

  ```
  systemctl stop firewalld
  ```

  禁用firewalld服务

  ```
  systemctl mask firewalld
  ```


## iptables使用

- 语法

  ```
  iptables [选项][参数]
  ```

- 选项

  ```
  -t,--table table 指定要操作的表，table必须是raw,nat,filter,mangle中的一个，默认filter表
  -A：向规则链中添加条目
  -D：从规则链中删除条目
  -I：向规则链中插入条目
  -R：替换规则链中的条目
  -L：显示规则链中已有的条目
  -F：清除规则链中已有的条目
  -Z：清空规则链中数据包计数器和字节计数器
  -N：创建新的用户自定义规则链
  -P：定义规则链中的默认目标
  -h：显示帮助信息
  -p：指定要匹配的数据包协议类型
  -s：指定要匹配的数据包源ip地址
  -j<目标>：指定要跳转的目标
  -i<网络接口>：指定数据包进入本机的网络接口
  -o<网络接口>：指定数据包要离开本机所使用的网络接口
  ```

- iptables命令选项输入顺序：

  ```
  iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作
  ```

- 表名包括：

  - raw：高级功能，如：网址过滤
  - mangle：数据包修改
  - net：地址转换，用于网关路由器
  - filter：包过滤，用于防火墙规则

- 规则链名包括：

  - INPUT链：处理输入数据包
  - OUTPUT链：处理输出数据包
  - PORWARD链：处理转发数据包
  - PREROUTING链：用于目标地址转换
  - POSTOUTING链：用于源地址转换

- 动作包括：

  - ACCEPT：接收数据包
  - DROP：丢弃数据包
  - REDIRECT：重定向、映射、透明代理
  - SNAT：源地址转换
  - DNAT：目标地址转换
  - MASQUERADE：IP伪装(NAT)，用于ADSL
  - LOG：日志记录


## iptable使用实例

- 清除已有规则

  ```shell
  iptables -F	#清空所有默认规则
  iptables -X	#清空所有自定义规则
  iptables -Z	#所有计数器归0
  ```

- 开放指定端口

  ```shell
  iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT	#允许本地回环接口(即运行本机访问本机)
  iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT	#允许已建立的或相关联的通行
  iptables -A OUTPUT -j ACCEPT	#允许所有本机向外的访问
  iptables -A INPUT -p tcp --dport 22 -j ACCEPT	#允许访问22端口
  iptables -A INPUT -j REJECT	#禁止其他未被允许的访问规则
  ```

- 屏蔽IP

  ```shell
  iptables -I INPUT -s 123.45.6.7 -j DROP	#屏蔽单个IP
  iptables -I INPUT -s 123.0.0.0/8 -j DROP	#封从123.0.0.1到123.255.255.254整个段
  iptables -I INPUT -s 124.45.0.0/16 -j DROP	#封从123.45.0.1到123.45.255.254整个段
  ```

- 查看已添加的iptables规则

  ```
  iptables -L -n -v
  ```

- 删除已添加的iptables规则

  显示所有iptables规则，并以序号标记

  ```
  iptables -L -n --line-numbers
  ```

  删除INPUT中序号为8的规则：

  ```
  iptables -D INPUT 8
  ```

