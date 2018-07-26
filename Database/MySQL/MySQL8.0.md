# MySQL8.0

## 授权

- 使用以前的方式进行授权：

  ```
  grant all privileges on *.* to root@"%" identified by "root" with grant option;
  ```

  > 报错：ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'identified by 'Root@123' whith grant option' at line 1 

- 8.0要先创建用户和设置密码，然后授权：

  创建用户：

  ```sql
  create user 'root'@'%' identified by 'password';
  ```

  授权：

  ```sql
  grant all privileges on *.* to 'root'@'%' with grant option;
  flush privileges;
  ```

  