1. 查看所有数据库
`show databases;`
2. 进入数据库
`use 数据库名字`

3. mysqldump常见使用

```sql
netstat -ln | grep mysql

mysqldump --socket=/data/mysql/mysql-socket/mysql.sock -unextcloud -ppansoft2021 nextcloud > nextcloud.sql

mysqldump --socket=/var/lib/mysql/mysql.sock -unextcloud -ppansoft2021 nextcloud > /home/user/nextcloud.sql

# 备份所有数据库
mysql -e "show databases;" -uroot -pPansoft2022 2>/dev/null | grep -Ev "information_schema|mysql|sys|performance_schema" | xargs mysqldump -uroot -pPansoft2022 --databases > mysql_dump_$(date +%F).sql
# 锁表备份
mysqldump  --set-gtid-purged=off -X --lock-all-tables -udatalake_user -pMN85avwojklVSCbG DATA_LAKE_META | gzip  > /home/app/test-2022-9-8-1.sql.gz

# 压缩备份数据库
mysqldump --set-gtid-purged=OFF -u4A_USER -pZDBQ9K6LZ5xcJnra OSPIam | gzip > /data/mysql_data_backup/OSPIam.sql.gz

# 备份表
mysqldump --set-gtid-purged=OFF -u4A_USER -pZDBQ9K6LZ5xcJnra 库名 库表
# 备份某个数据库下所有表结构，没有数据
mysqldump -u4A_USER -pZDBQ9K6LZ5xcJnra -d 库名

# 备份某个数据库下某个表的表结构，没有数据
mysqldump -u4A_USER -pZDBQ9K6LZ5xcJnra -d 库名 表名

# 备份某个数据库下所有数据，不要表结构
mysqldump -u4A_USER -pZDBQ9K6LZ5xcJnra -t 库名

# 备份某个数据库下某个表的所有数据，不要表结构
mysqldump -u4A_USER -pZDBQ9K6LZ5xcJnra -t 库名 表名
```

参考文档：
<https://www.cnblogs.com/DBA-3306/p/12632978.html>

4. 查看MYSQL数据库中所有用户

```SQL
SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;
```

5. 查看 mysql 版本
mysql -V

6. 删除数据库
`DROP DATABASE [ IF EXISTS ] <数据库名>`

7. 导入数据库

> 创建数据库
> 1、首先建空数据库
> mysql>create database abc;
> 创建数据库时指定字符集
> create database mydb character set utf-8;
> 2、导入数据库
> 方法一：
> （1）选择数据库
> mysql>use abc;
> （2）设置数据库编码
> mysql>set names utf8;
> （3）导入数据（注意sql文件的路径）
> mysql>source /home/abc/abc.sql;
> 方法二：
> mysql -u用户名 -p密码 数据库名 < 数据库名.sql
> mysql -uabc_f -p abc < abc.sql

8. 创建数据库

```sql
create database if not exists nextcloudPro character set utf8mb4 collate utf8mb4_general_ci;

```

9. 创建用户

```sql
create user 'nextcloudPro'@'%' identified by 'Pansoft2021@';

create user 'nextcloudPro'@'%' identified by 'Pansoft2021@';
```

10. 赋予用户相应操作数据库的权限

```sql
grant all privileges on nextcloudPro.* to 'nextcloudPro'@'%';
```

11. 刷新让配置生效
flush privileges;

12. 列出表字段

```sql
show columns from database_name.table_name
```

13. 删除用户

```sql
drop user 'jack'@'%';
```

14. 删除数据库内所有表

```sql
SELECT concat('DROP TABLE IF EXISTS ', table_name, ';')
FROM information_schema.tables
WHERE table_schema = '数据库名字';


SELECT concat('DROP TABLE IF EXISTS ', table_name, ';')
FROM information_schema.tables
WHERE table_schema = 'nextcloudPro';
```

15. 赋予 root 用户远程登录的权限
```sql
grant all privileges on *.* to root@"%" identified by "password" with grant option;
flush privileges;
```

16. 删除数据库

```sql
drop database <数据库名>;
```

17. 查看数据库内用户

```sql
SELECT user FROM mysql.user;
```

18. 远程登录数据库

```shell
mysql -h 192.168.5.116 -P 3306 -u root -p123456
```

19. 查看指定数据库字符集

```sql
select schema_name,default_character_set_name from information_schema.schemata where schema_name = '数据库名';

# 进入数据库
# 查看当前数据库库字符集
SHOW VARIABLES LIKE 'character_set_database';
# 
SHOW VARIABLES LIKE 'character_set_database';
```

20. 查看数库大小

```
select concat(round(sum(data_length/1024/1024),2),'MB') as data from information_schema.tables where table_schema='OSPSPRING';
```

21. mysqldump 导出导入数据
<https://www.jianshu.com/p/c3d8366326c1>
<https://www.cnblogs.com/qq78292959/p/3637135.html>

22. 登录 mysql

```sql
mysql -h 192.168.5.116 -P 3306 -u root -p123456 
```

23. 查看某个用户拥有的权限

```sql
show grants for 'dolphin'@'%';
```

24. 赋予某个用户权限

```sql
GRANT SELECT, LOCK TABLES ON `dolphin`.* TO 'dolphin'@'%';
```

25. 删除赋予用户的权限

```sql
REVOKE  SELECT, LOCK TABLES ON `dolphin`.* from 'dolphin'@'%';
```

26. 某个数据库的所有表
```sql
select * from information_schema.tables where table_schema = 'zavier';

show tables;
```

27. 修改用户名密码
方法一：通过sql命令修改密码
命令格式：set password for 用户名@localhost = password('新密码'); 
新版本mysql 命令：alter user 用户名@localhost identified by '新密码';

28. 更新某个字段值
```sql
UPDATE <表名> SET 字段 1=值 1 [,字段 2=值 2… ] [WHERE 子句 ]
[ORDER BY 子句] [LIMIT 子句]
```

29. mysql 运行时间
```shell
show global status like 'uptime';
```

30. 查看缓冲池大小
```
show variables like 'innodb_buffer_pool_size'\G;
# 用 GB 为单位显示缓冲池大小
select @@innodb_buffer_pool_size/1024/1024/1024;
```

31. 显示 innodb 引擎相关参数
```
SHOW ENGINE INNODB STATUS
```

32. 查看用户所有权限
```
show grants for 'backup'@'localhost'\G
```

33. 删除用户
```
mysql&gt; use mysql;
Database changed
mysql&gt; delete from user where user='root' and host='%' ;
mysql&gt;flush privileges ;
Query OK, 1 row affected (0.02 sec)
```

34. mysql 连接情况
```
show full processlist;
show processlist;
select * from information_schema.processlist;
```

35. 查看报错类型
```sql
select * from performance_schema.host_cache\G;
```

36. 删除用户权限
```
REVOKE ALL PRIVILEGES ON *.* FROM 'datalake_user'@'%';
```

https://blog.csdn.net/liaowenxiong/article/details/120612060

37. 清空查询缓存
```sql
reset query cache;
```

38. 查看 数据库 信息
```
show create database 数据库名称;
```

39. 查看数据库创建信息
```
show create database <databasename>;
```

40. 获取某个表前几条数据
select * from table_name limit 3;

41. 创建表
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    ...
);

42. 对某个表添加字段
```sql
ALTER TABLE table_name ADD column_name column_type;
```

43. 删除符合条件的数据
```sql
DELETE FROM students WHERE graduation_year = 2021;
```

44. 删除所有行
```sql
DELETE FROM orders;
```

45. 修改表字段类型
```sql
ALTER TABLE table_name MODIFY column_name column_type DEFAULT NULL COMMENT '注释'; 
```

46. 后台执行导入数据库
```shell
nohup /data/mysql-8.4.0-linux-glibc2.28-x86_64/bin/mysql -uroot -p'IwN3SWAs]UDe' --database=db_DF62690101 < /data/db_DF62690101.sql &
```
