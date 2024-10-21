# 主从部署
## 适用于 MySQL 5.7 版本
mysql 配置文件

```bash
cat << EOF > /etc/my.cnf
[client]
socket=/data/mysql-5.7.35/mysql.sock
[mysqld]
port=3306
basedir=/data/mysql-5.7.35
datadir=/data/mysql-5.7.35/data
socket=/data/mysql-5.7.35/mysql.sock
user=mysql
character-set-server=utf8mb4
server-id=1
gtid-mode=ON
enforce-gtid-consistency=ON
log-bin=mysql-bin
binlog_format=row
expire_logs_days=7
open_files_limit=10240
max_connections=1000
max_connect_errors=1000
max_allowed_packet=16M
thread_cache_size=300
key_buffer_size=256M
table_open_cache=1024
query_cache_type=1
query_cache_size=128M
join_buffer_size=8M
sort_buffer_size=8M
read_buffer_size=2M
read_rnd_buffer_size=2M
binlog_cache_size=12M
max_binlog_size=512M
innodb_buffer_pool_size=40G
innodb_log_file_size=512M
innodb_log_buffer_size=32M
#innodb_flush_log_at_trx_commit=2
sync_binlog=1
innodb_flush_log_at_trx_commit=1
slave-skip-errors=1062,1146  # 主从同步跳过常见错误
EOF
```
## 创建同步用户


## 主从配置

修改 my.cnf 配置
[mysqld]下面增加下面配置
```sql
server-id=1
log-bin = mysql-bin
binlog_format = ROW
binlog_row_image = minimal
```

## 登录主节点授权.输入从节点 账户 登录密码

```sql
grant replication slave on *.* to 'root'@' ' identified by ' ';
flush privileges;

# 查看主节点状态

show master status\G
```

## mysql 从节点 my.cnf 配置修改

```sql
# 在[mysqld]加入下面的内容：
 
# 服务的唯一编号
server-id = 2
# 开启mysql binlog功能
log-bin = mysql-bin
# binlog记录内容的方式，记录被操作的每一行
binlog_format = ROW
# 减少记录日志的内容，只记录受影响的列
binlog_row_image = minimal
```

## mysql 从节点登录同步主节点信息

```sql
# 设置主服务器ip，同步账号密码，同步位置
Change master to master_host='192.168.45.67',master_user='root',master_password='password@123',master_log_file='mysql-bin.000001',master_log_pos=592,master_port=3306;
# 开启同步功能
start slave;
```

## 查看从节点状态

```sql
show slave status\G
```

注意：Slave_IO_Running和Slave_SQL_Running的状态都为Yes时，说明从库配置成功。

## 测试主从功能
create database testdata;

## 清除主从
### 清除从
```sql
mysql>stop slave;

QueryOK, 0 rowsaffected (0,00 sec)
mysql>reset slave all;

QueryOK, 0 rowsaffected (0,04 sec)

mysql> show slave status\G

Emptyset (0,00 sec)

```
此时真正实现了清除slave同步复制关系！

### 清除主
```sql
mysql> reset master;
mysql> show master status\G;
```

---
参考文档：
<https://blog.csdn.net/weixin_37998428/article/details/120999548>
[清除主从](https://www.cnblogs.com/musen/p/11162783.html)


# 主主同步 + keepalived
主主同步就是 交叉主从同步
修改 my.cnf 配置文件
主配置文件中要有
```
server_id = 1
log-bin = mysql-bin
binlog_format = mixed
relay-log = relay-bin
relay-log-index = slave-relay-bin.index
auto-increment-increment = 2
auto-increment-offset = 1
```
从配置文件
```
server_id = 2
log-bin = mysql-bin
binlog_format = mixed
relay-log = relay-bin
relay-log-index = slave-relay-bin.index
auto-increment-increment = 2
auto-increment-offset = 2
```

## 2、将mysql1设为mysql2的主服务器
### mysql1 创建授权账户，允许在 mysql2 主机上连接
```
grant replication slave on *.* to 'doublemaster'@'10.240.53.135' identified by '2U3_8zZfVy6Mqfwd@';
# 查看当前 mysql1 状态
show master status\G;
```
在mysql2上将mysql1设为自已的主服务器并开启slave功能：
```
change master to master_host='10.240.53.133', master_user='doublemaster', master_password='2U3_8zZfVy6Mqfwd@', master_log_file='mysql.000004', master_log_pos=34758,master_port=13306;
start slave;
```
## 3、将 mysql2 设为 mysql1 的主服务器
### mysql2 创建授权账户，允许在 mysql1 主机上连接

```
grant replication slave on *.* to 'doublemaster'@'10.240.53.133' identified by '2U3_8zZfVy6Mqfwd@';
# 查看当前 mysql1 状态
show master status\G;
```
在mysql2上将mysql1设为自已的主服务器并开启slave功能：
```
change master to master_host='10.240.53.135', master_user='doublemaster', master_password='2U3_8zZfVy6Mqfwd@', master_log_file='mysql.000004', master_log_pos=34758,master_port=13306;
start slave;
```
## 4、测试主主同步
mysql1 上进行增删改查操作后，在 mysql2 上查看是否生效。
mysql2 上进行增删改查操作后，在 mysql1 上查看是否生效。

## Keepalived 部署
### 1. Keepalived 安装
```
yum install -y keepalived
```
### 2. 添加防火墙富贵则
```
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.100.153" accept"
```
### 3. 修改 keepalived 配置文件




问题
Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.
物理网卡绑定 ip 需要注意网卡名称搜索 (ip add 提供的名字不一定为真) bond0.1015@bond0 中 bond0.1015 为真实网卡名称
```
ll /etc/sysconfig/network-scripts 
ll /proc/sys/net/ipv4/conf
ifcofnig
```


(VI_1): received an invalid passwd!

---
参考文章：
https://cloud.tencent.com/developer/article/1343127#:~:text=Keepaliv,%E8%AF%81%E7%B3%BB%E7%BB%9F%E7%9A%84%E9%AB%98%E5%8F%AF%E7%94%A8%E3%80%82

