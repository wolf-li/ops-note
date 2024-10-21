## 一.执行存储过程/函数报错账号不存在

### 问题描述：执行存储过程报错

```sql
mysql>call create_no_by_day('STUDENT','CREATE_TIME');
ERROR 1449 (HY000):The user specified as a definer ('TEST_111'@'172.%.%.%') does not exist
```

### 分析

这情况是因为当时机器上这存储过程是由用户’TEST_111’@'172.%.%.%‘创建，但是存储过程导入到了新的机器后，这新机器没有这个用户而报错。

### 三种解决办法

1、重建这个存储过程，把definer那段代码取消。
2、在新机器上建立这个用户’TEST_111’@‘172.%.%.%’。
3、修改定义者，替换所有存储过程是这个定义者的为新机器已有账号。

### 解决：修改定义者

1.查看机器不存在这个账号
```sql
select host,user from mysql.user where host='172.%.%.%' and user='TEST_111';
```

2.替换所有存储过程是这个定义者的为新机器已有账号。
```sql
UPDATE  mysql.proc set DEFINER='TEST_222@172.%.%.%' where DEFINER='TEST_111@172.%.%.%';
```

## 二.遇到DDL变更的时候发生阻塞

### 问题描述

添加字段、添加索引等DDL语句时候会被阻塞，show processlist 会看到显示 Waiting for table metadata lock. 后续的对这些表的select也会被阻塞

### 分析

autocommit=0，怀疑有未提交事务，导致产生了元数据锁。

### 解决

1.如果打开了performance_schema 监控，通过语句定位未提交事务：
```sql
SELECT
locked_schema,
locked_table,
locked_type,
waiting_processlist_id,
waiting_age,
waiting_query,
waiting_state,
blocking_processlist_id,
blocking_age,
substring_index(sql_text,“transaction_begin;” ,-1) AS blocking_query,
sql_kill_blocking_connection
FROM
(
SELECT
b.OWNER_THREAD_ID AS granted_thread_id,
a.OBJECT_SCHEMA AS locked_schema,
a.OBJECT_NAME AS locked_table,
“Metadata Lock” AS locked_type,
c.PROCESSLIST_ID AS waiting_processlist_id,
c.PROCESSLIST_TIME AS waiting_age,
c.PROCESSLIST_INFO AS waiting_query,
c.PROCESSLIST_STATE AS waiting_state,
d.PROCESSLIST_ID AS blocking_processlist_id,
d.PROCESSLIST_TIME AS blocking_age,
d.PROCESSLIST_INFO AS blocking_query,
concat('KILL ', d.PROCESSLIST_ID) AS sql_kill_blocking_connection
FROM
performance_schema.metadata_locks a
JOIN performance_schema.metadata_locks b ON a.OBJECT_SCHEMA = b.OBJECT_SCHEMA
AND a.OBJECT_NAME = b.OBJECT_NAME
AND a.lock_status = ‘PENDING’
AND b.lock_status = ‘GRANTED’
AND a.OWNER_THREAD_ID <> b.OWNER_THREAD_ID
AND a.lock_type = ‘EXCLUSIVE’
JOIN performance_schema.threads c ON a.OWNER_THREAD_ID = c.THREAD_ID
JOIN performance_schema.threads d ON b.OWNER_THREAD_ID = d.THREAD_ID
) t1,
(
SELECT
thread_id,
group_concat( CASE WHEN EVENT_NAME = ‘statement/sql/begin’ THEN “transaction_begin” ELSE sql_text END ORDER BY event_id SEPARATOR “;” ) AS sql_text
FROM
performance_schema.events_statements_history
GROUP BY thread_id
) t2
WHERE
t1.granted_thread_id = t2.thread_id
```

2.杀进程
```sql
kill blocking_processlist_id
```

3.没有打开wait/lock/metadata/sql/mdl情况下，针对sleep进程执行kill
```sql
SELECT concat(‘kill ‘,processlist_id,’;’)
FROM performance_schema.events_statements_current a JOIN performance_schema.threads b USING(thread_id)
JOIN information_schema.processlist c ON b.processlist_id = c.id
WHERE a.sql_text NOT LIKE ‘%performance%’ and command=‘sleep’ order by c.time desc;
```

## 三.批量更新数据卡住

### 问题描述

批量更新很慢，没添加索引，添加索引被阻塞

### 分析

事务不自动提交容易造成元数据锁冲突

### 解决

执行
```sql
SELECT concat(‘kill ‘,processlist_id,’;’)
FROM performance_schema.events_statements_current a JOIN performance_schema.threads b USING(thread_id)
JOIN information_schema.processlist c ON b.processlist_id = c.id
WHERE a.sql_text NOT LIKE ‘%performance%’ and command=‘sleep’ order by c.time desc;
```
杀进程

## 四.一个环境数据库占用空间满情况

### 问题描述：一个环境数据库空间已满

### 分析：数据没有定时清理，没有监控报警。一个保存实时消息的大表上百亿数据占用过大

问题解决：
1.能登陆mysql情况下，truncate table 大表(无用数据，可清除)，回收空间
2.不能登陆mysql情况下，删除部分binlog日志，让mysql启动起来，再清理其他数据。

## 五.数据库迁移

### 问题描述：已备份了数据，想把数据库导进另一个环境，但是想换一个数据库名字，以免把另一个环境的同名数据库覆盖

问题解决：
1.导出数据

# mysqldump -uroot -pxxx_001 --set-gtid-purged=OFF admin > admin.sql

2． 建新库
mysql>create database admin1;

3． 建立账号，授权
mysql>CREATE USER admin1@'%' identified by 'Admin1_!@#';
mysql>GRANT ALL PRIVILEGES ON  admin1.* TO admin1@'%';
mysql>flush privileges;

4． 导入数据到新库admin1

# mysql -uroot -pxxx_001 admin1 <admin.sql

这样，就把原库admin导出的表数据，导入了新库admin1里面。新账号admin1已授权访问新库admin1,新账号密码是Admin1_!@#

注： 操作2,3是sql语句，在mysql里面执行。 操作1,4是在linux命令行上操作。

## 六.数据库日志出现多个断连记录

### 问题描述：
数据库日志出现多个断连记录，显示为Got an error writing communication packets

### 分析：
有可能是客户端异常退出了，应用重启，也有可能是网络链路异常。这种提示一般是[NOTE],属于提示信息

问题解决：
关闭警报
set global log_warning=1;
另：若是出现Got timeout reading communication packets或者Got timeout writing communication packets，属于客户端的空连接时间过长，超过了wait_timeout和interactive_timeout的时间，可以调整wait_timeout/interactive_timeout

## 七.非法断电造成mysql启动报错

### 问题描述：
非法断电造成mysql数据损坏

### 分析：
突然断电造成缓存数据丢失，跟已刷盘的数据不一致。需要重做从库

问题解决：到主库物理备份数据，恢复到从库，恢复主从同步。
物理备份恢复过程：
主库:
mysqlbackup --user=root --password=xxx_001 --backup-dir=/mysql/data/backup --backup-image=./dball.mbi --with-timestamp --compress-level=9 --compress-method=zlib --skip-binlog --skip-relaylog backup-to-image #备份数据
scp dball.mbi root@192.168.137.111:/mysql/data/backup/ #拷贝到目标机器backup目录

从库:
cd /mysql/data/backup/
chown mysql.mysql dball.mbi
su - mysql
mysql.server stop
cd /mysql/data/undo
rm -rf *#清空undo日志
cd /mysql/data/redo
rm -rf* #清空redo日志
cd /mysql/data/dbs
rm -rf * #清空数据
mysqlbackup --backup-image=/mysql/data/backup/dball.mbi --backup-dir=/mysql/data/backup --uncompress copy-back-and-apply-log --force #恢复数据
mysql.server start #启动mysql
mysql -uroot -p
reset slave all; #重置
change master to master_host = ‘192.168.137.110’, master_port = 3310, master_user = ‘rpl_user’, master_password = ‘rpl_001’, master_auto_position=1 ;#设置主从同步复制
start slave;#启动同步
show slave status\G;#查看同步状况

## 八.非法断电造成mysql同步复制无法启动

### 问题描述：
relay报错，error:look foring afer relay.000013

### 分析：
relay得到的gtid比执行的gtid少，得到的部分gtid丢失

show slave status\G;
Master_UUID: 37be0d7b-e11e-11e9-bafb-fa163e9dcbee
Retrieved_Gtid_Set: 37be0d7b-e11e-11e9-bafb-fa163e9dcbee:2290-2302(少)
Executed_Gtid_Set: 37be0d7b-e11e-11e9-bafb-fa163e9dcbee:1-2352(大),
5324e653-f0c2-11e9-84d0-fa163e897a41:1-363,
5661dce0-e11e-11e9-ab09-fa163e897a41:1-318

问题解决：
调整gtid，以relay获得的gtid为准，重做主从同步
reset slave all;#重置
reset master;#重置
SET @@GLOBAL.GTID_PURGED=‘37be0d7b-e11e-11e9-bafb-fa163e9dcbee:1-2302(少),5324e653-f0c2-11e9-84d0-fa163e897a41:1-363,5661dce0-e11e-11e9-ab09-fa163e897a41:1-318’;#以relay获得的gtid为准，设置GTID_PURGED
change master to master_host = ‘192.168.137.110’, master_port = 3310, master_user = ‘rpl_user’, master_password = ‘rpl_001’, master_auto_position=1 ;#设置主从同步复制
start slave; #启动同步
show slave status\G;#查看同步状况

## 九.mysql压力测试，插入时间增大，压不上去

### 问题描述：
压力测试，数据库插入出现延时情况

### 分析：
沟通发现，mysql是个人安装，非标准化，设置不当，例如32G的内存，innodb_buffer_pool_size只默认128M

问题解决：
1.动态在线增大innodb_buffer_pool_size值。
set global innodb_buffer_pool_size=1610241024*1024;

2.为了永久生效，在my.cnf里面设置
innodb_buffer_pool_size=16G
innodb_buffer_pool_instances = 8
重启mysql

## 十.通过恢复文件加载空间恢复表数据

### 问题描述：
出现测试环境数据库ibdata1损坏，无法启动

### 分析：
通过开发环境的mysql全量备份，恢复到测试环境。由于开发环境的admin库数据库结构跟测试环境的admin库一样，但是数据不一样，需要用原测试环境数据文件恢复原表数据

问题解决：
1.备份数据
mkdir backup
cp dbs/admin/*backup/
2.开发环境mysqlbackup全量备份恢复到测试环境
3.卸载库admin所有表空间
mysql -uroot -p -e “select concat(‘alter table ‘,table_name,’ DISCARD tablespace;’) from information_schema.TABLES where table_schema=‘admin’ and table_type=‘BASE TABLE’;” >admin_discard.sql
vi admin_discard.sql
删除第一行concat(‘alter table ‘,table_name,’ DISCARD tablespace;’)
mysql -uroot -p admin <admin_discard.sql
4.拷贝回.ibd文件
cp -f backup/*.ibd dbs/admin/
5.加载库admin所有表空间
mysql -uroot -p -e “select concat(‘alter table ‘,table_name,’ IMPORT tablespace;’) from information_schema.TABLES where table_schema=‘admin’ and table_type=‘BASE TABLE’;” >admin_import.sql
vi admin_import.sql
删除第一行concat(‘alter table ‘,table_name,’ IMPORT tablespace;’)
mysql -uroot -p admin <admin_import.sql
6.启动mysql
mysql.server start

## 十一.XA事务未提交，更新数据超时

### 问题描述：出现更新数据超时问题

### 分析：查看事务进程，发现有未提交的xa事务，thread_id为0

### 问题解决：

1.先列出xa事务
```sql
mysql> xa recover;
±---------±-------------±-------------±---------------------------------------------------------------------------------------------------------------------------------+
| formatID | gtrid_length | bqual_length | data |
±---------±-------------±-------------±---------------------------------------------------------------------------------------------------------------------------------+
| 3072 | 64 | 64 | PRBMPAPP4/c734a63e-211f-4202-8aa9-78d2/174064445 3 |
±---------±-------------±-------------±---------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.10 sec)
xa回滚的格式是xa rollback ‘left(data,gtrid_length)’,‘substr(data,gtrid_length+1,bqual_length)’,formatid;

```
2.转16制
```sql
mysql> set session autocommit=1;
Query OK, 0 rows affected (0.00 sec)

mysql> xa recover convert xid;
±---------±-------------±-------------±-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| formatID | gtrid_length | bqual_length | data |
±---------±-------------±-------------±-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 3072 | 64 | 64 | 0x5052424D50415050342F63373334613633652D323131662D343230322D386161392D373864322F3137343036343434350000000000000000000000000000000033000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000 |
±---------±-------------±-------------±-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
3.回滚事务
mysql> xa rollback 0x5052424D50415050342F63373334613633652D323131662D343230322D386161392D373864322F31373430363434343500000000000000000000000000000000,0x33000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000,3072;
Query OK, 0 rows affected (0.01 sec)
```

## 十二.mysql升级5.7.26，更换驱动后原账号连不上

### 问题描述：
mysql从5.7.24升级到mysql5.7.26，驱动从mysql connector java更换为mariadb，业务升级前就是这个密码，运行很久，有3套mysql，有1套重启数据库还不能恢复，有2套重启数据库恢复了

### 分析：
mariadb驱动对sha256_password识别不好，改用低版本mysql_native_password

### 问题解决：
1.查询当前密码加密方式
```sql
SELECT user, host, plugin FROM mysql.user;
mysql> SELECT user, host, plugin FROM mysql.user;
±--------------±--------------±----------------------+
| user | host | plugin |
±--------------±--------------±----------------------+
| root | % | sha256_password |
| mysql.session | localhost | mysql_native_password |
| mysql.sys | localhost | mysql_native_password |
| mysql_om | % | sha256_password |
| rpl_user | 10.110.10.19 | sha256_password |
| root | 10.110.10.156 | sha256_password |
| xxxadmin | % | sha256_password |
| xxxuser | % | sha256_password |
| mysql | % | sha256_password |
±--------------±--------------±----------------------+
14 rows in set (0.00 sec)

```
2.修改加密方式
```sql
ALTER USER ‘xxxuser’@’%’ IDENTIFIED WITH mysql_native_password BY ‘XXX_1234’;
flush privileges;
```

## 十三.mysql无法启动，数据目录丢失

### 问题描述： 
mysql无法启动，查看error日志，提示mysql.user不存在，查看my.cnf数据所在目录，发现目录里面除了ibdata1和ib_logfile，auto.cnf，没有其他数据

### 分析：
挂载mysql数据库目录丢失。mount的盘信息没写入fstab里面，断电重启后，没有挂载磁盘

## 问题解决：
1.挂载数据库目录
mount -t ext3 /dev/sda3 /mysqldata
2.启动mysql
service mysql start

## 十四.无法登陆mysql，can’t connect。。。。mysql.sock

### 问题描述：
can’t connect 。。。。。。’/mysqldata/mysql/mysql.sock’

### 分析：
在数据目录找不到mysql.sock文件或者mysql没有启动

### 问题解决：
1.找不到mysql.sock文件
登录加上-S 参数 或者ln -s 建立软连或者在my.cnf里面写上
mysql -uroot -p -S /usr/mysql.sock
或者 ln -s /usr/mysql.sock /mysqldata/mysql/mysql.sock
2.mysql没有启动
mysql.server start

## 十五.数据目录存在非法目录

### 问题描述：
错误日志显示mysql error invalid (old ) table or database name ‘lost+found’

### 分析：
datadir里面的一个目录代表一个数据库，非法写入目录，导致mysql不能识别

### 问题解决：
删除目录’lost+found’。

## 十六.错误日志报警告信息长度不匹配

### 问题描述：
[Warning] InnoDB: Table mysql/innodb_table_stats has length mismatch in the column name table_name. Please run mysql_upgrade

### 分析：
5.7.17：table_name varchar(64) COLLATE utf8_bin NOT NULL

5.7.24：table_name varchar(199) COLLATE utf8_bin NOT NULL,
可以看出，5.7.24 版本上的 innodb_index_stats 和innodb_tables_stats 的 table_name 列，长度从64 变成了 199，而升级后的5.7.24中 table_name 还是64

### 问题解决：
1.执行更新
mysql_upgrade -u root -p –force
2.重启mysqld进程
mysql.server start

## 十七.错误日志报警告信息ignored in --skip-name-resolve mode

### 问题描述：
[Warning] ‘user’ entry ‘mysql.session@localhost’ ignored in --skip-name-resolve mode

### 分析：
跳过dns解析，打开了skip_name_resolve后出现

### 问题解决：
删除不必要的账号
delete from mysql.user where user=‘mysql.session’

## 十八.大量Waiting in connection_control plugin连接

### 问题描述:
show processlist;结果超过2000条结果. 90%连接:Waiting in connection_control plugin

### 分析：
启动了防爆破插件connection_control

### 问题解决:
show plugins;
select * from information_schema.CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS;
1.修改参数show variables like ‘%connection_control%’;
```sql
set global connection_control_failed_connections_threshold=0;
set global connection_control_max_connection_delay=3000;
```

或者2.重启mysql服务
mysql.server restart

3.kill进程
select concat('kill ', id, ‘;’) from information_schema.processlist where command= ‘Waiting in connection_control plugin’;

## 十九.备库同步复制通道channel值为空
### 现象描述：
多源复制，同步复制通道channel值为空
### 问题分析：
需要重置复制通道
### 问题解决:
1.停止slave
stop slave;
2.清空同步复制关联
reset slave all;
3.重置同步复制关联
change master to master_host = ‘192.168.137.110’, master_port = 3310, master_user = ‘rpl_user’, master_password = ‘xxxxxxxxxx’, master_auto_position=1 for channel ‘rpl1’;
4.启动slave
start slave;

另外，多源同步复制通道的知识点，官网和互联网上都很少，以下是经过本人实践的：
1.多源复制，同步复制通道channel修改。
1）update update mysql.slave_master_info set channel_name=“新通道名”;
2）service mysqld restart
3）show slave status\G; #这时候会多一条通道channel，旧通道和新通道
4）stop slave for channel “旧通道名”;
5）reset slave all for channel “旧通道名”;

2.多源复制，同步复制通道channel多一条，删除
1）stop slave for channel “删除通道名”;
2）reset slave all for channel “删除通道名”;#若是通道名为空，值"".
3）show slave status\G;

## 二十.mysql时区跟系统数据不一致

### 问题描述

mysql> select now();
±--------------------+
| now() |
±--------------------+
| 2020-05-27 06:27:24 |
±--------------------+
1 row in set (0.00 sec)
mysql> show global variables like ‘%time_zone%’;
±-----------------±-------+
| Variable_name | Value |
±-----------------±-------+
| system_time_zone | UTC |
| time_zone | SYSTEM |
±-----------------±-------+
2 rows in set (0.01 sec)
mysql@tb6usrdb2:~> date
Wed May 27 09:27:34 CEST 2020

### 分析：系统是CEST时区，与mysql的时区不一致，差3个小时

### 问题解决：
1.修改mysql时区：
set time_zone=’+2:00’; #+2,不是+3

2.结果：
mysql> show global variables like ‘%time_zone%’;
±-----------------±-------+
| Variable_name | Value |
±-----------------±-------+
| system_time_zone | UTC |
| time_zone | +02:00 |
±-----------------±-------+
2 rows in set (0.00 sec)

若是要永久生效就是在my.cnf中修改，添加default-time_zone=’+2:00’
然后重启mysql。
mysql> show global variables like ‘%time_zone%’;
±-----------------±-------+
| Variable_name | Value |
±-----------------±-------+
| system_time_zone | CEST |
| time_zone | +02:00 |
±-----------------±-------+
2 rows in set (0.00 sec)

## 二十一导出数据库报错

### 现象：
mysqldump: Error: 'Access denied; you need (at least one of) the PROCESS privilege(s) for this operation' when trying to dump tablespaces

### 解决方法

<https://anothercoffee.net/how-to-fix-the-mysqldump-access-denied-process-privilege-error/>

## 二十二mysqldump 普通用户导出有问题

### 现象：
mysqldump: Error: ‘Access denied； you need (at least one of) the PROCESS privilege(s) for this opera

### 解决方法
```shell
# mysql -uroot -p
```

```sql
mysql> GRANT PROCESS ON *.* TO 'demo'@'localhost';
mysql> flush privileges;
```
process 权限是可以查看所有用户线程连接的权限

---
参考文档：
https://www.cnblogs.com/kerrycode/p/7421777.html


## 二十三 Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'
```sql
# master 端
flush logs;
show master status\G;

# slave 端
stop slave;
change master to MASTER_LOG_FILE='mysql-bin.000003',MASTER_LOG_POS=154;
start slave;
```


## 二十四 The user specified as a definer ('sdc'@'%') does not exist
```sql
sql> select concat("alter DEFINER=`root`@`localhost` SQL SECURITY DEFINER VIEW `",TABLE_SCHEMA,"`.",TABLE_NAME," as ",VIEW_DEFINITION,";") from information_schema.VIEWS where DEFINER = 'sdc@%';
```
生成的 sql 执行即可；
https://www.jianshu.com/p/dd8bca0fedc8
https://www.cnblogs.com/jiangwz/p/12666431.html

## 二十四 ERROR 1010 (HY000): Error dropping database (can't rmdir './test/', errno: 17)
在删除数据库的时候报标题所示错误

mysql> drop database test;
ERROR 1010 (HY000): Error dropping database (can't rmdir './test/', errno: 17)
 

问题原因：

test目录下存在着MySQL数据库不知道的文件，即MySQL数据库中没有该文件的数据字典信息。

如下所示，

[root@localhost data]# cd /usr/local/mysql-advanced-5.6.23-linux-glibc2.5-x86_64/data/test
[root@localhost test]# ls
123
 

解决方法：

手动删除test目录下的该文件

# rm -rf 123
登录数据库，重新执行drop database操作

mysql> drop database test;
Query OK, 0 rows affected (0.01 sec)