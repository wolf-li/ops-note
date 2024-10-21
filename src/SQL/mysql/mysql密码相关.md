# MySQL 密码相关问题
## MySQL/MariaDB忘记root密码的简单解决方法

PS：如果你万一忘记了MySQL的root密码，下面是重设密码的最简单最安全的方法了，操作不影响数据。

1、vim /etc/my.cnf，在[mysqld]字段（一定要放在这里，否则无效！）加入skip-grant-tables配置，意思就是跳过密码验证。

2、重启MySQL服务service mysqld restart，用mysql -u root直接回车空密码登录进去。

3、重设MySQL的root新密码：
在重置密码前，先要重载授权表：
```sql
mysql> FLUSH PRIVILEGES;
```

MySQL 5.7.6+：
```sql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```
MySQL 5.7.6以及之后的版本，使用ALTER USER语法来修改密码。

MySQL 5.7.5以及之前的版本

```sql
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('MyNewPass');
```
MySQL 5.7.5以及之前的版本使用SET PASSWORD语法修改密码。

如果上面的方法修改密码有错，可以直接修改mysql.user表：

```sql
UPDATE mysql.user SET authentication_string = PASSWORD('MyNewPass')
WHERE User = 'root' AND Host = 'localhost';
FLUSH PRIVILEGES;
```
注意：mysql5.7 user表里已经去掉了password字段，改为了authentication_string。


4、刷新数据库：flush privileges; 后退出。

5、现在可以用新密码 !@#123 登录了！

说明：此方法适用于MySQL5.0-5.5各版本，也适用于MariaDB。

---
参考文档：
<http://www.ha97.com/5197.html>
https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html

## MySQL 修改密码及密码强度规则

### MySQL 修改密码方式
1. 使用 set password 命令修改
2. 使用 mysqladmin 修改
3. 使用 update 修改 user 表
4. 使用 alert 重置密码

### 修改密码规则


---
参考文档：
<https://blog.csdn.net/weixin_42150196/article/details/108386851>
