# MySQL定时备份（全量备份+增量备份）

## MySQL 全量备份

参考 zone7_ 的 实战-MySQL定时备份系列文章

参考 zmcyu 的 mysql数据库的完整备份、差异备份、增量备份

更多binlog的学习参考马丁传奇的 MySQL的binlog日志，这篇文章写得认真详细，如果看的认真的话，肯定能学的很好的。
如果查看binlog是出现语句加密的情况，参考 mysql row日志格式下 查看binlog sql语句

说明
产品上线后，数据非常非常重要，万一哪天数据被误删，那么就gg了，准备跑路吧。
所以要对线上的数据库定时做全量备份和增量备份。

增量备份的优点是没有重复数据，备份量不大，时间短。但缺点也很明显，需要建立在上次完全备份及完全备份之后所有的增量才能恢复。

MySQL没有提供直接的增量备份方法，但是可以通过mysql二进制日志间接实现增量备份。二进制日志对备份的意义如下：

二进制日志保存了所有更新或者可能更新数据的操作
二进制日志在启动MySQL服务器后开始记录，并在文件达到所设大小或者收到flush logs 命令后重新创建新的日志文件
只需定时执行flush logs 方法重新创建新的日志，生成二进制文件序列，并及时把这些文件保存到一个安全的地方，即完成了一个时间段的增量备份。
全量备份
mysqldump --lock-all-tables --flush-logs --master-data=2 -u root -p test > backup_sunday_1_PM.sql
参数 --lock-all-tables
对于InnoDB将替换为 --single-transaction。
该选项在导出数据之前提交一个 BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于事务表，例如 InnoDB 和 BDB。本选项和 --lock-tables 选项是互斥的，因为 LOCK TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用 --quick 选项。

参数 --flush-logs，结束当前日志，生成并使用新日志文件

参数 --master-data=2，该选项将会在输出SQL中记录下完全备份后新日志文件的名称，用于日后恢复时参考，例如输出的备份SQL文件中含有：CHANGE MASTER TO MASTER_LOG_FILE='MySQL-bin.000002', MASTER_LOG_POS=106;

参数 test，该处的test表示数据库test，如果想要将所有的数据库备份，可以换成参数 --all-databases

参数 --databases 指定多个数据库

参数 --quick或-q，该选项在导出大表时很有用，它强制 MySQLdump 从服务器查询取得记录直接输出而不是取得所有记录后将它们缓存到内存中。

参数 --ignore-table，忽略某个数据表，如 --ignore-table test.user 忽略数据库test里的user表

更多mysqldump 参数，请参考网址

全量备份脚本shell

```
#!/bin/bash
# mysql 数据库全量备份
 
# 用户名、密码、数据库名
username="root"
password="tencns152"
dbName="goodthing"
 
beginTime=`date +"%Y年%m月%d日 %H:%M:%S"`
# 备份目录
bakDir=/home/mysql/backup
# 日志文件
logFile=/home/mysql/backup/bak.log
# 备份文件
nowDate=`date +%Y%m%d`
dumpFile="${dbName}_${nowDate}.sql"
gzDumpFile="${dbName}_${nowDate}.sql.tgz"

cd $bakDir
# 全量备份（对所有数据库备份，除了数据库goodthing里的village表）
/usr/local/mysql/bin/mysqldump -u${username} -p${password} --quick --events --databases ${dbName} --ignore-table=goodthing.village --ignore-table=goodthing.area --flush-logs --delete-master-logs --single-transaction > $dumpFile
# 打包
/bin/tar -zvcf $gzDumpFile $dumpFile
/bin/rm $dumpFile
 
endTime=`date +"%Y年%m月%d日 %H:%M:%S"`
echo 开始:$beginTime 结束:$endTime $gzDumpFile succ >> $logFile
 
# 删除所有增量备份
cd $bakDir/daily
/bin/rm -f *
 ```

这里全量备份只备份了一个数据库，因为如果所有数据库都备份的话，文件太大了。这里的取舍我也不是很清楚，毕竟自己还在学习阶段，没有实际的操作经验。

## 增量备份

1. 检查log_bin是否开启
进入mysql命令行，执行 show variables like '%log_bin%';

```sql
mysql> show variables like '%log_bin%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin                         | OFF   |
| log_bin_basename                |       |
| log_bin_index                   |       |
| log_bin_trust_function_creators | OFF   |
| log_bin_use_v1_row_events       | OFF   |
| sql_log_bin                     | ON    |
+---------------------------------+-------+
6 rows in set (0.01 sec)
```

如上所示，log_bin 未开启；如果log_bin开启，则跳过第2步，直接进入第3步。

2. 开启 log_bin，并重启mysql
编辑 mysql 的配置文件 vim /etc/my.cnf，在 mysqld 下面添加下面2条配置
[mysqld]
log-bin=/var/lib/mysql/mysql-bin
server_id=152
Tip1: 一定要加 server_id，否则会报错。至于server_id的值，随便设就可以。
Tip2: log_bin 中间可以下划线_相连，也可以-减号相连。同理server_id也一样。

重启mysql
service mysqld restart
再次在mysql命令行中执行 show variables like '%log_bin%'

```sql
mysql> show variables like '%log_bin%';
+---------------------------------+--------------------------------+
| Variable_name                   | Value                          |
+---------------------------------+--------------------------------+
| log_bin                         | ON                             |
| log_bin_basename                | /var/lib/mysql/mysql-bin       |
| log_bin_index                   | /var/lib/mysql/mysql-bin.index |
| log_bin_trust_function_creators | OFF                            |
| log_bin_use_v1_row_events       | OFF                            |
| sql_log_bin                     | ON                             |
+---------------------------------+--------------------------------+
6 rows in set (0.01 sec)
 ```

3. 备份
进入mysql命令行，执行 show master status;

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      430 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```

当前正在记录日志的文件名是 mysql-bin.000003

比如当前数据库test的bk_user只有2条记录

```sql
mysql> select * from test.bk_user;
+----+------+------+------+
| id | name | sex  | age  |
+----+------+------+------+
|  1 | 小明 | 男   |   25 |
|  2 | 小红 | 女   |   21 |
+----+------+------+------+
2 rows in set (0.00 sec)

```

插入一条新的记录

```sql
mysql> insert into test.bk_user(name, sex, age) values('小强', '男', 24);
Query OK, 1 row affected (0.02 sec)
mysql> select * from test.bk_user;
+----+------+-----+-----+
| id | name | sex | age |
+----+------+-----+-----+
|  1 | 小明 | 男  |  25 |
|  2 | 小红 | 女  |  21 |
|  5 | 小强 | 男  |  24 |
+----+------+-----+-----+
3 rows in set (0.03 sec)

```

执行命令mysqladmin -uroot -p密码 flush-logs，生成并使用新的日志文件
再次查看当前使用的日志文件，已经变为 mysql-bin.000004 了。
mysql-bin.000003 则记录着刚才执行的 insert 语句的日志。

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000004 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```

到这里，其实已经完成了增量备份了。

恢复增量备份
首先假装误删数据库记录

```sql
mysql> delete from test.bk_user where id=4;
Query OK, 1 row affected (0.01 sec)

mysql> select * from test.bk_user;
+----+------+------+------+
| id | name | sex  | age  |
+----+------+------+------+
|  1 | 小明 | 男   |   25 |
|  2 | 小红 | 女   |   21 |
+----+------+------+------+
2 rows in set (0.00 sec)
```

从备份的日志文件mysql-bin.000003中恢复数据

```bash
[root@centos56 ~]# mysqlbinlog --no-defaults /var/lib/mysql/mysql-bin.000003 | mysql -uroot -p test
Enter password:
ERROR 1032 (HY000) at line 36: Can't find record in 'bk_user'
```

如果你也遇到这个问题的话，不妨修改 /etc/my.cnf 配置试试。
我在server_id那一行下添加了 slave_skip_errors=1032 ，然后就执行成功了，不再报错。

```sql
mysql> select * from test.bk_user;
+----+------+------+------+
| id | name | sex  | age  |
+----+------+------+------+
|  1 | 小明 | 男   |   25 |
|  2 | 小红 | 女   |   21 |
|  5 | 小强 | 男   |   24 |
+----+------+------+------+
3 rows in set (0.00 sec)
```

增量备份的shell脚本

```bash
#!/bin/bash
 
# 增量备份时复制mysql-bin.00000*的目标目录，提前手动创建这个目录
BakDir=/home/mysql/backup/daily
# 日志文件
LogFile=/home/mysql/backup/bak.log
 
# mysql的数据目录
BinDir=/var/lib/mysql-bin
# mysql的index文件路径，放在数据目录下的
BinFile=/var/lib/mysql-bin/mysql-bin.index
 
# 这个是用于产生新的mysql-bin.00000*文件
/usr/local/mysql/bin/mysqladmin -uroot -ptencns152 flush-logs
 
Counter=`wc -l $BinFile | awk '{print $1}'`
NextNum=0
# 这个for循环用于比对$Counter,$NextNum这两个值来确定文件是不是存在或最新的
for file in `cat $BinFile`
do
        base=`basename $file`
        NextNum=`expr $NextNum + 1`
        if [ $NextNum -eq $Counter ]
        then
                echo $base skip! >> $LogFile
        else
                dest=$BakDir/$base
                #test -e用于检测目标文件是否存在，存在就写exist!到$LogFile去
                if(test -e $dest)
                then
                        echo $base exist! >> $LogFile
                else
                        cp $BinDir/$base $BakDir
                        echo $base copying >> $LogFile
                fi
        fi
done
 
echo `date +"%Y年%m月%d日 %H:%M:%S"` $Next Bakup succ! >> $LogFile
```

定时备份
执行命令 crontab -e，添加如下配置

```bash
# 每个星期日凌晨3:00执行完全备份脚本
0 3 * * 0 /bin/bash -x /root/bash/Mysql-FullyBak.sh >/dev/null 2>&1
 
# 周一到周六凌晨3:00做增量备份
0 3 * * 1-6 /bin/bash -x /root/bash/Mysql-DailyBak.sh >/dev/null 2>&1
```

遇到的问题
Can't connect to local MySQL server through socket '/tmp/mysql.sock'
mysqladmin: connect to server at 'localhost' failed
error: 'Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)'
Check that mysqld is running and that the socket: '/tmp/mysql.sock' exists
去修改mysql的配置文件，添加

[mysqladmin]

# 修改为相应的sock

socket=/var/lib/mysql/mysql.sock
执行mysqldump时遇到 Unknown table 'column_statistics' in information_schema (1109)
[root@centos56 bash]# /usr/local/mysql/bin/mysqldump -uroot -ptencns152 --quick --events --all-databases --flush-logs --delete-master-logs --single-transaction > /home/mysql/backup/1.sql  
mysqldump: [Warning] Using a password on the command line interface can be insecure.
mysqldump: Couldn't execute 'SELECT COLUMN_NAME,                       JSON_EXTRACT(HISTOGRAM, '$."number-of-buckets-specified"')                FROM information_schema.COLUMN_STATISTICS                WHERE SCHEMA_NAME = 'atd' AND TABLE_NAME = 'box_info';': Unknown table 'column_statistics' in information_schema (1109)
如果使用MySQL 8.0+版本提供的命令行工具mysqldump来导出低于8.0版本的MySQL数据库到SQL文件，会出现Unknown table 'column_statistics' in information_schema的错误，因为早期版本的MySQL数据库的information_schema数据库中没有名为COLUMN_STATISTICS的数据表。

解决问题的方法是，使用8.0以前版本MySQL附带的mysqldump工具，最好使用待备份的MySQL服务器版本对应版本号的mysqldump工具，mysqldump可以独立运行，并不依赖完整的MySQL安装包，比如在Windows中，可以直接从MySQL安装目录的bin目录中将mysqldump.exe复制到其他文件夹，甚至从一台电脑复制到另一台电脑，然后在CMD窗口中运行。

当前使用是的MySQL 5.7.22。把5.7.20的 MYSQL_HOME/bin/mysqldump 替换掉 5.7.22的，接着就能顺利执行mysqldump了，也真是奇了怪了。

---
参考文章：
<https://www.cnblogs.com/haicheng92/p/10106517.html#:~:text=%20MySQL%E6%B2%A1%E6%9C%89%E6%8F%90%E4%BE%9B%E7%9B%B4%E6%8E%A5%E7%9A%84%E5%A2%9E%E9%87%8F%E5%A4%87%E4%BB%BD%E6%96%B9%E6%B3%95%EF%BC%8C%E4%BD%86%E6%98%AF%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87mysql%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%97%A5%E5%BF%97%E9%97%B4%E6%8E%A5%E5%AE%9E%E7%8E%B0%E5%A2%9E%E9%87%8F%E5%A4%87%E4%BB%BD%E3%80%82%20%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%97%A5%E5%BF%97%E5%AF%B9%E5%A4%87%E4%BB%BD%E7%9A%84%E6%84%8F%E4%B9%89%E5%A6%82%E4%B8%8B%EF%BC%9A,%E5%8F%AA%E9%9C%80%E5%AE%9A%E6%97%B6%E6%89%A7%E8%A1%8Cflush%20logs%20%E6%96%B9%E6%B3%95%E9%87%8D%E6%96%B0%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84%E6%97%A5%E5%BF%97%EF%BC%8C%E7%94%9F%E6%88%90%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%96%87%E4%BB%B6%E5%BA%8F%E5%88%97%EF%BC%8C%E5%B9%B6%E5%8F%8A%E6%97%B6%E6%8A%8A%E8%BF%99%E4%BA%9B%E6%96%87%E4%BB%B6%E4%BF%9D%E5%AD%98%E5%88%B0%E4%B8%80%E4%B8%AA%E5%AE%89%E5%85%A8%E7%9A%84%E5%9C%B0%E6%96%B9%EF%BC%8C%E5%8D%B3%E5%AE%8C%E6%88%90%E4%BA%86%E4%B8%80%E4%B8%AA%E6%97%B6%E9%97%B4%E6%AE%B5%E7%9A%84%E5%A2%9E%E9%87%8F%E5%A4%87%E4%BB%BD%E3%80%82>
<https://blog.csdn.net/weixin_42377364/article/details/120065016>


## mysqlpump 备份


---
参考文章
https://www.cnblogs.com/zhoujinyi/p/5684903.html
