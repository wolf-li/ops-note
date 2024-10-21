版本为postgresql -11
一、基础环境
通过环境部署两个单机的postgresql服务

主机	192.168.23.129	5432
从机	192.168.23.130	5432



## 二、主库配置

### 1.进入数据库
```
切换到postgres用户（安装好生成默认的用户）
su - postgres
进入数据库
psql
```
### 2.创建账号并授权用来同步数据,只赋予登录和复制的权限即可
```
# 这里设置账号为postgresql 密码为 Pansoft2021
 
postgres=# create role postgresql login replication encrypted password 'dNvg%*Pp&X';

create role repl login replication encrypted password 'w@hYEkYu&u';
```
### 3.修改配置文件
```
①修改映射配置文件
vim /data/pgsql/11/data/pg_hba.conf
  
 # 在IPv4下添加
 
 
host all     all       0.0.0.0/0             md5
  
 # 附:也可添加以下内容指定复制账号 root 为用户名
 -- host replication       root            0.0.0.0/0               md5

```

trust为无密码信任登录，只需输入ip和port即可登录；mds需要用户验证登录；ident为映射系统账户到pgsql访问账户。

②修改归档配置
vi /var/lib/pgsql/11/data/postgresql.conf
  
```
# 配置修改的内容
listen_addresses = '*'
wal_level = hot_standby #热备模式
max_wal_senders= 6 #可以设置最多几个流复制链接，差不多有几个从，就设置多少
wal_keep_segments = 10240 #重要配置
max_connections = 512 #从库的 max_connections要大于主库
archive_mode = on #允许归档
archive_command = 'cp %p /url/path%f' #根据实际情况设置
```

三、从库配置
1.拷贝数据库
```
# 切换数据库用户
su - postgres
 
 
# 如果从库的data库已经初始化,需要删除掉
rm -rf /data/pgsql/11/data/*
 
# 传输data数据
pg_basebackup -h 192.168.23.129 -U postgresql -D /data/pgsql/11/data -X stream -P
 
# 复制同步恢复文件
cp /data/pgsql/11/share/recovery.conf.sample /data/pgsql/11/data/recovery.conf
```

2.修改恢复配置文件

vim /data/pgsql/11/data/recovery.conf

```
# 修改内容
recovery_target_timeline = 'latest'   #同步到最新数据
standby_mode = on          #指明从库身份
trigger_file = 'failover.now'
primary_conninfo = 'host=192.168.63.134 port=5432 user=repl password=123456'   #连接到主库信息
```

3.修改postgresql.conf配置文件
 vi /data/pgsql/11/data/postgresql.conf
 

```
listen_addresses = '*'            # what IP address(es) to listen on;
port = 5432                # (change requires restart)
max_connections = 1000            # (change requires restart)
shared_buffers = 128MB            # min 128kB
dynamic_shared_memory_type = posix    # the default is the first option
wal_level = replica        # minimal, replica, or logical
archive_mode = on        # enables archiving; off, on, or always
archive_command = 'cp %p /var/lib/pgsql/12/data/pg_archive/%f'        # command to use to archive a logfile segment
wal_sender_timeout = 60s    # in milliseconds; 0 disables
hot_standby = on            # "on" allows queries during recovery
max_standby_streaming_delay = 30s    # max delay before canceling queries
wal_receiver_status_interval = 10s    # send replies at least this often
hot_standby_feedback = on        # send info from standby to prevent
log_directory = 'log'    # directory where log files are written,

```
四、验证
1.登录主库
 select client_addr,sync_state from pg_stat_replication;


---
参考文档：
https://www.cnblogs.com/hope123/p/11958820.html
