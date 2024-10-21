
## Cassandra3 数据库集群部署

### 1、环境准备（三台机器）

```
10.20.11.22
10.20.11.25
10.20.11.28
```

**下载 Cassandra 安装包**

地址：https://archive.apache.org/dist/cassandra/3.11.8/apache-cassandra-3.11.8-bin.tar.gz

**下载 JDK** （要求 1.8）

下载路径：https://www.oracle.com/cn/java/technologies/javase/javase-jdk8-downloads.html#license-lightbox



### 2、**安装 JDK**

将下载好的压缩包上传到服务器

```shell
# 解压
tar -zxvf jdk-8u271-linux-x64.tar.gz
```

添加环境变量

```shell
echo "export JAVA_HOME=/data/jdk1.8.0_301
export JRE_HOME=/data/jdk1.8.0_301/jre
export CLASSPATH=.:\$JAVA_HOME/lib:\$JRE_HOME/lib:\$CLASSPATH
export PATH=\$JAVA_HOME/bin:\$JRE_HOME/bin:\$PATH" >> /etc/profile
source /etc/profile
java -version
```

```
java version "1.8.0_271"
Java(TM) SE Runtime Environment (build 1.8.0_271-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.271-b09, mixed mode)
```



### 3、**安装 Cassandra**

将下载好的安装包上传至服务器挂载目录

```shell
# 解压
tar -xzvf apache-cassandra-3.11.8-bin.tar.gz 
```

**配置Cassandra环境变量**

```shell
echo "CASSANDRA_HOME=/data/apache-cassandra-3.11.8
PATH=\$PATH:\$CASSANDRA_HOME/bin
export CASSANDRA_HOME PATH" >> /etc/profile.d/cassandra.sh
source /etc/profile.d/cassandra.sh
```

创建cassandra 数据存放的文件夹
```shell
mkdir -p /data/cassandra/{data,commitlog,saved_caches}
```

创建 cassandra 用户，并赋权

```shell
#添加用户
useradd -M cassandra -s /sbin/nologin
#设置密码
echo "cassandra" | passwd --stdin cassandra

#将Cassandra的安装目录（解压目录）赋权给新用户
chown -R cassandra: /data/apache-cassandra-3.11.8/
chown -R cassandra: /data/cassandra/
```

修改 `cassandra.yaml` 配置文件

```shell
cd $CASSANDRA_HOME/conf 
```

```yaml
cluster_name: 'Test Cluster' #更改为自己的集群名
commitlog_directory: /data/cassandra/commitlog #更改为自己的路径，存放commitlog
data_file_directories:
    - /data/cassandra/data #更改为自己的路径，数据文件的存放路径
saved_caches_directory: /data/cassandra/saved_caches #更改为自己的路径，缓存存放目录
listen_address: 10.20.11.22 #更改为本机ip地址
rpc_address: 10.20.11.22 #更改为本机ip地址
broadcast_rpc_address: 10.20.11.22 #本机ip地址
start_rpc: true  #开启 thrift rpc 服务(默认是 false)
seed_provider:
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          - seeds: "10.20.11.22,10.20.11.25,10.20.11.28" #设置集群种子节点IP，如果多个用逗号分隔
 
memtable_heap_space_in_mb: 7440 #这里配置最大使用的内存空间数量
concurrent_reads: 16  #建议采用16*磁盘数
concurrent_writes: 32  #建议8*cpu核心数

#修改用户名密码配置，将默认的值修改为
authenticator: PasswordAuthenticator   
authorizer: CassandraAuthorizer
```

修改 `cqlsh.py` 文件

```shell
# 在/bin 目录下
DEFAULT_HOST = '10.20.11.22'  //修改成自己ip
```



**继续配置其他两台机器，除了主机IP不同外，其余的都相同**

**继续配置其他两台机器，除了主机IP不同外，其余的都相同**



**启动 Cassandra**（先启动**seeds**节点  再启动非seeds节点）

任意目录下输入 **cassandra**(每台机器都要操作)，如果最后出现如下几行则表示启动成功

> 测试阶段建议使用 `cassandra -f`  启动，区别是`cassandra -f` 在前台启动，按Ctrl + C可停止

```shell
INFO  [main] 2021-08-24 16:36:36,260 Gossiper.java:1723 - No gossip backlog; proceeding
```

查看集群各节点状态使用命令 **nodetool status**

```shell
[cassandra@i-455bTFx83-9 apache-cassandra-3.11.8]$ ./bin/nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address       Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.240.58.66  208.83 KiB  256          72.5%             26d5a9de-4eb5-4387-b3b4-9d2382e988f7  rack1
UN  10.240.58.67  281.77 KiB  256          63.4%             02f09825-c2eb-4689-b6cd-1a99f02dc600  rack1
UN  10.240.58.68  140.56 KiB  256          64.1%             f728e297-0efd-498b-a8d2-f5097f7a9383  rack1

```

可以看到IP地址前面都是 **UN** ，说明三台机器启动成功且运行正常(不正常显示 **DN**)。



### 4、验证

在任意目录键入 `cqlsh`，进入数据库命令行工具，接下来可进行数据验证操作：

> 如果配置了修改用户名密码，使用 cqlsh -u cassandra -p cassandra <ip> <port> ，默认端口9042

创建一个**键空间**（相当于一个数据库）

```sql
CREATE KEYSPACE test_keyspace WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};
```

> 上面命令创建了名为 test_keyspace 的 keyspace；并且采用 SimpleStrategy 进行副本复制。如果是生产环境，建议将副本因子（replication factor）设置为 3。

查看所有的键

```sql
DESCRIBE KEYSPACES;
```

使用新键并创建一个表

```sql
cqlsh> USE test_keyspace;  
cqlsh:test_keyspace> CREATE TABLE test_user (first_name text , last_name text, PRIMARY KEY (first_name)) ;
```

往表中插入数据

```sql
cqlsh:test_keyspace> INSERT INTO test_user (first_name, last_name) VALUES ('test', 'Hadoop');

cqlsh:test_keyspace> INSERT INTO test_user (first_name, last_name) VALUES ('Zhang', 'San');

cqlsh:test_keyspace> INSERT INTO test_user (first_name) VALUES ('Wu');
```

然后在集群中分别查询数据，可以发现都已经同步过去了

```sql
cqlsh:test_keyspace> SELECT * FROM test_user;
```



### 5、更改用户名和密码

Cassandra 默认用户名密码都为 `cassandra`，不安全，所以我们需自己创建用户。

**修改 `cassandra.yaml` 配置文件**，上面配置已经修改

将默认的

```yaml
authenticator: AllowAllAuthenticator
```

修改为

```yaml
authenticator: PasswordAuthenticator  #决定是否继续用户登录
```

将

```yaml
authorizer: AllowAllAuthorizer
```

修改为

```yaml
authorizer: CassandraAuthorizer  #通过授权决定登录之后用户具有哪些权限
```



**启动 cassandra**

使用 `cql` 连接，cassandra 是默认账户和密码，默认端口 9042

```shell
cqlsh -u cassandra -p cassandra <ip> <port>
```

创建用户

```sql
CREATE USER root WITH PASSWORD '123456' SUPERUSER ;
```

> 这里用户有两种，一个是**superuser**，一种是**nosuperuser**
>

查看所有用户

```sql
LIST USERS; 
```

退出`cqlsh`，用新用户登陆

```
# 删除默认用户
drop user cassandra;
```

> 如果是集群模式，只需要在一台机器上配置就可以了，会自动同步过去



***配置完成！！***

vi /etc/systemd/system/cassandra.service
```
[Unit]
Description=Cassandra Server Service
After=network.service
 
[Service]
Type=simple
# 需要注意jdk安装目录
Environment=JAVA_HOME=/data/jdk1.8.0_301
 
PIDFile=/data/apache-cassandra-3.11.10/cassandra.pid
User=cassandra
Group=cassandra
# 此处为Cassandra包解压后的路径
ExecStart=/data/apache-cassandra-3.11.10/bin/cassandra -f -p /data/apache-cassandra-3.11.10/cassandra.pid
StandardOutput=journal
StandardError=journal
LimitNOFILE=100000
LimitMEMLOCK=infinity
LimitNPROC=32768
LimitAS=infinity
 
[Install]
WantedBy=multi-user.target

```

防火墙
```
firewall-cmd --zone=public --add-port=7000/tcp --permanent
firewall-cmd --zone=public --add-port=7001/tcp --permanent
firewall-cmd --zone=public --add-port=9042/tcp --permanent
firewall-cmd --zone=public --add-port=9160/tcp --permanent
firewall-cmd --reload
```


参考文档：
https://www.jianshu.com/p/9f76dc4f0c3e


## 报错
### 进入 cassandra 报错
现象
```
Unauthorized: Error from server: code=2100 [Unauthorized] message="Only superusers can create a role with superuser status"
```
解决
修改配置文件



### 忘记密码