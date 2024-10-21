## MongoDB分片集群简介  
在单机环境下，高频率的查询会给服务器 CPU 和 I/O 带来巨大的负担，基于这个原因，MongoDB 提供了分片机制用于解决大数据集的分布式部署，从而提高系统的吞吐量。一个标准的 MongoDB 分片集群通常包含以下三类组件：  
1. mongos，数据库集群请求的入口，所有的请求都通过mongos进行协调，不需要在应用程序添加一个路由选择器，mongos自己就是一个请求分发中心，它负责把对应的数据请求转发到对应的shard服务器上。在生产环境通常有多mongos作为请求的入口，防止其中一个挂掉所有的mongodb请求都没有办法操作。  
2. config server，为配置服务器，存储所有数据库元信息（路由、分片）的配置。mongos本身没有物理存储分片服务器和数据路由信息，只是缓存在内存里，配置服务器则实际存储这些数据。mongos第一次启动或者关掉重启就会从 config server 加载配置信息，以后如果配置服务器信息变化会通知到所有的 mongos 更新自己的状态，这样 mongos 就能继续准确路由。在生产环境通常有多个 config server 配置服务器，因为它存储了分片路由的元数据，防止数据丢失！  
3. shard，分片（sharding）是指将数据库拆分，将其分散在不同的机器上的过程。将数据分散到不同的机器上，不需要功能强大的服务器就可以存储更多的数据和处理更大的负载。基本思想就是将集合切成小块，这些块分散到若干片里，每个片只负责总数据的一部分，最后通过一个均衡器来对各个分片进行均衡（数据迁移）。  
分片集群部署规划  
  
节点IP、组件及端口规划：  
  
主机名 | IP地址 | mongos-router | config-server | shard  
----|------|---------------|---------------|------  
mongodb01 | 192.168.92.80 | mongos-router:27017 | config-server:27018 | shard1:27019 shard2:27020 shard3:27021  
mongodb02 | 192.168.92.81 | mongos-router:27017 | config-server:27018 | shard1:27019 shard2:27020 shard3:27021  
mongodb03 | 192.168.92.82 | mongos-router:27017 | config-server:27018 | shard1:27019 shard2:27020 shard3:27021  
  
configServer配置服务器建议部署为包含3个成员的副本集模式，出于测试目的，您可以创建一个单成员副本集；  
shard分片请使用至少包含三个成员的副本集。出于测试目的，您可以创建一个单成员副本集；  
mongos没有副本集概念，可以部署1个、2个或多个。  
本次部署使用3台服务器，部署3个mongos、3个configServer、以及3个分片，每个分片包含3个成员，都分布在不同服务器上。  
#### 修改主机名  
```  
hostnamectl set-hostname mongodb01  
hostnamectl set-hostname mongodb02  
hostnamectl set-hostname mongodb03  
```  
## 在所有主机上进行下列操作  


#### 配置域名解析  
```  
cat > /etc/hosts <<EOF  
192.168.92.80 mongodb01  
192.168.92.81 mongodb02  
192.168.92.82 mongodb03  
EOF  
```  
#### mongodb 需要的依赖包  
`yum install -y libcurl openssl xz-libs`  
#### 配置环境变量  
```  
echo "PATH=$PATH:/data/mongodb/bin" > /etc/profile.d/mongodb.sh  
source /etc/profile.d/mongodb.sh  
```  
## 1. 在其中任意一台下载软件包，解压缩  
```  
sudo wget -P /data https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.0.26.tgz  
sudo tar xzf /data/mongodb-linux-x86_64-rhel70-4.0.26.tgz -C /data  
sudo mv /data/mongodb-linux-x86_64-rhel70-4.0.26.tgz /data/mongodb  
sudo rm -f /data/mongodb-linux-x86_64-rhel70-4.0.26.tgz  
```  
## 2. 创建相关目录和文件  
1）创建数据目录和日志目录  
mkdir -p /data/mongodb/cluster/{config,mongos,shard1,shard2,shard3}/{data,logs}  
2）创建配置文件  
  
configServer配置文件  
```  
cat > /data/mongodb/cluster/config/mongod.conf <<EOF  
systemLog:  
  destination: file  
  logAppend: true  
  path: /data/mongodb/cluster/config/logs/mongod.log  
  
# Where and how to store data.  
storage:  
  dbPath: /data/mongodb/cluster/config/data  
  journal:  
    enabled: true  
  
# how the process runs  
processManagement:  
  fork: true  # fork and run in background  
  pidFilePath: /data/mongodb/cluster/config/mongod.pid  # location of pidfile  
  timeZoneInfo: /usr/share/zoneinfo  
  
# network interfaces  
net:  
  port: 27018  
  bindIp: mongodb01  
  
sharding:  
  clusterRole: configsvr  
  
replication:  
  replSetName: config  
EOF  
```  
mongos-router配置文件，注意mongos没有storage部分配置：  
```  
cat > /data/mongodb/cluster/mongos/mongod.conf <<EOF  
systemLog:  
  destination: file  
  logAppend: true  
  path: /data/mongodb/cluster/mongos/logs/mongod.log  
  
# how the process runs  
processManagement:  
  fork: true  # fork and run in background  
  pidFilePath: /data/mongodb/cluster/mongos/mongod.pid  # location of pidfile  
  timeZoneInfo: /usr/share/zoneinfo  
  
# network interfaces  
net:  
  port: 27017  
  bindIp: mongodb01  
  
sharding:  
  configDB: config/mongodb01:27018,mongodb02:27018,mongodb03:27018  
EOF  
```  
  
shard1分片配置文件  
```  
cat > /data/mongodb/cluster/shard1/mongod.conf <<EOF  
systemLog:  
  destination: file  
  logAppend: true  
  path: /data/mongodb/cluster/shard1/logs/mongod.log  
  
# Where and how to store data.  
storage:  
  dbPath: /data/mongodb/cluster/shard1/data  
  journal:  
    enabled: true  
  
# how the process runs  
processManagement:  
  fork: true  # fork and run in background  
  pidFilePath: /data/mongodb/cluster/shard1/mongod.pid  # location of pidfile  
  timeZoneInfo: /usr/share/zoneinfo  
  
# network interfaces  
net:  
  port: 27019  
  bindIp: mongodb01  
  
sharding:  
    clusterRole: shardsvr  
      
replication:  
    replSetName: shard1  
EOF  
  
```  
shard2分片配置文件  
```  
cat >  /data/mongodb/cluster/shard2/mongod.conf <<EOF  
systemLog:  
  destination: file  
  logAppend: true  
  path: /data/mongodb/cluster/shard2/logs/mongod.log  
  
# Where and how to store data.  
storage:  
  dbPath: /data/mongodb/cluster/shard2/data  
  journal:  
    enabled: true  
  
# how the process runs  
processManagement:  
  fork: true  # fork and run in background  
  pidFilePath: /data/mongodb/cluster/shard2/mongod.pid  # location of pidfile  
  timeZoneInfo: /usr/share/zoneinfo  
  
# network interfaces  
net:  
  port: 27020  
  bindIp: mongodb01  
  
sharding:  
    clusterRole: shardsvr  
      
replication:  
    replSetName: shard2  
EOF  
```  
shard3分片配置文件  
```  
cat > /data/mongodb/cluster/shard3/mongod.conf <<EOF  
systemLog:  
  destination: file  
  logAppend: true  
  path: /data/mongodb/cluster/shard3/logs/mongod.log  
  
# Where and how to store data.  
storage:  
  dbPath: /data/mongodb/cluster/shard3/data  
  journal:  
    enabled: true  
  
# how the process runs  
processManagement:  
  fork: true  # fork and run in background  
  pidFilePath: /data/mongodb/cluster/shard3/mongod.pid  # location of pidfile  
  timeZoneInfo: /usr/share/zoneinfo  
  
# network interfaces  
net:  
  port: 27021  
  bindIp: mongodb01  
  
sharding:  
    clusterRole: shardsvr  
      
replication:  
    replSetName: shard3  
EOF  
```  
使用 scp 传递给其他机器  
`scp -r /data/mongodb root@ip:/data/`  
  
在 mongodb02 节点更新配置文件绑定地址  
`grep -rl bindIp /data/mongodb/cluster/ | xargs sed -i 's#bindIp: mongodb01#bindIp: mongodb02#g'`  
在 mongodb03 节点更新配置文件绑定地址  
`grep -rl bindIp /data/mongodb/cluster/ | xargs sed -i 's#bindIp: mongodb01#bindIp: mongodb03#g'`  
验证更新成功  
`grep -rn "bindIp: mongodb0" /data/mongodb/cluster/`  
## 3 启动并初始化configserver  
连接3个节点执行以下命令分别启动configServer  
  
`mongod -f /data/mongodb/cluster/config/mongod.conf`  
  
连接到第一个节点mongodb01  
  
[root@mongodb01 ~]# mongo --host mongodb01 --port 27018  
  
初始化configServer副本集  
  
```  
rs.initiate(  
  {  
    _id: "config",  
    configsvr: true,  
    members: [  
      { _id : 0, host : "mongodb01:27018" },  
      { _id : 1, host : "mongodb02:27018" },  
      { _id : 2, host : "mongodb03:27018" }  
    ]  
  }  
)  
```  
  
查看节点状态  
  
`rs.status()`  

返回结果
```
 "members" : [
                {
                        "_id" : 0,
                        "name" : "mongodb1:27018",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 618,
                        "optime" : {
                                "ts" : Timestamp(1628591796, 1),
                                "t" : NumberLong(47)
                        },
                        "optimeDate" : ISODate("2021-08-10T10:36:36Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "syncing from: mongodb3:27018",
                        "electionTime" : Timestamp(1628591794, 1),
                        "electionDate" : ISODate("2021-08-10T10:36:34Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "mongodb2:27018",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 16,
                        "optime" : {
                                "ts" : Timestamp(1628591796, 1),
                                "t" : NumberLong(47)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1628591796, 1),
                                "t" : NumberLong(47)
                        },
                        "optimeDate" : ISODate("2021-08-10T10:36:36Z"),
                        "optimeDurableDate" : ISODate("2021-08-10T10:36:36Z"),
                        "lastHeartbeat" : ISODate("2021-08-10T10:36:42.056Z"),
                        "lastHeartbeatRecv" : ISODate("2021-08-10T10:36:42.531Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "mongodb1:27018",
                        "syncSourceHost" : "mongodb1:27018",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 1
                },
                {
                        "_id" : 2,
                        "name" : "mongodb3:27018",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 7,
                        "optime" : {
                                "ts" : Timestamp(1628591796, 1),
                                "t" : NumberLong(47)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1628591796, 1),
                                "t" : NumberLong(47)
                        },
                        "optimeDate" : ISODate("2021-08-10T10:36:36Z"),
                        "optimeDurableDate" : ISODate("2021-08-10T10:36:36Z"),
                        "lastHeartbeat" : ISODate("2021-08-10T10:36:41.817Z"),
                        "lastHeartbeatRecv" : ISODate("2021-08-10T10:36:41.034Z"),
                        "pingMs" : NumberLong(1),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "mongodb1:27018",
                        "syncSourceHost" : "mongodb1:27018",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 1
                }
```

启动并初始化shard分片  
3个节点每个节点执行以下命令启动3个shard分片副本集  
  
```  
mongod -f /data/mongodb/cluster/shard1/mongod.conf  
mongod -f /data/mongodb/cluster/shard2/mongod.conf  
mongod -f /data/mongodb/cluster/shard3/mongod.conf  
```  
  
连接到第一个shard分片并初始化  
  
[root@mongodb01 ~]# mongo --host mongodb01 --port 27019  
  
```  
  
rs.initiate(  
  {  
    _id: "shard1",  
    members: [  
      { _id : 0, host : "mongodb01:27019" },  
      { _id : 1, host : "mongodb02:27019" },  
      { _id : 2, host : "mongodb03:27019" }  
    ]  
  }  
)  
  
```  
  
连接到第二个shard分片并初始化  
  
`[root@mongodb01 ~]# mongo --host mongodb01 --port 27020`  
  
  
```  
rs.initiate(  
  {  
    _id: "shard2",  
    members: [  
      { _id : 0, host : "mongodb01:27020" },  
      { _id : 1, host : "mongodb02:27020" },  
      { _id : 2, host : "mongodb03:27020" }  
    ]  
  }  
)  
  
```  
  
  
  
连接到第三个shard分片并初始化  
  
`[root@mongodb01 ~]# mongo --host mongodb01 --port 27021`  
  
  
```  
rs.initiate(  
  {  
    _id: "shard3",  
    members: [  
      { _id : 0, host : "mongodb01:27021" },  
      { _id : 1, host : "mongodb02:27021" },  
      { _id : 2, host : "mongodb03:27021" }  
    ]  
  }  
)  
  
```  
  
查看各个分片副本集节点状态  
  
`rs.status()`  
  
启动并初始化mongos  
3个节点启动mongos-router，注意启动时使用mongos命令而非mongod命令：  
  
`mongos -f /data/mongodb/cluster/mongos/mongod.conf`  
  
连接到第一个mongos节点mongodb01并初始化  
  
`[root@mongodb01 ~]# mongo --host mongodb01 --port 27017`  
  
执行以下操作将3个分片副本集添加到集群：  
  
```  
sh.addShard( "shard1/mongodb01:27019,mongodb02:27019,mongodb03:27019")  
sh.addShard( "shard2/mongodb01:27020,mongodb02:27020,mongodb03:27020")  
sh.addShard( "shard3/mongodb01:27021,mongodb02:27021,mongodb03:27021")  
```  
  
  
查看mongos状态  
  
`sh.status()`  
  
查看当前节点运行的所有mongod进程：  
  
[root@mongodb01 ~]# ps -ef | grep mongod | grep -v grep  
root      37847      1  5 17:53 ?        00:01:43 mongod -f /data/mongodb/cluster/config/mongod.conf  
root      37993      1  3 18:01 ?        00:00:49 mongod -f /data/mongodb/cluster/shard1/mongod.conf  
root      38036      1  3 18:01 ?        00:00:48 mongod -f /data/mongodb/cluster/shard2/mongod.conf  
root      38079      1  3 18:01 ?        00:00:50 mongod -f /data/mongodb/cluster/shard3/mongod.conf  
root      38329      1  0 18:13 ?        00:00:06 mongos -f /data/mongodb/cluster/mongos/mongod.conf  
  
  
至此分片集群部署完成，接下来执行数据分片操作。  
  
为数据库启用分片  
在分片集合之前，必须为集合的数据库启用分片，连接任意一台mongos节点shell，以mongodb01节点为例。  
  
`mongo --host mongodb01 --port 27017`  
  
创建testdb数据库  
  
`use testdb;`  
  
为集合启用分片  
为testdb数据库启用分片  
  
`sh.enableSharding("testdb")`  
  
为order集合设置分片规则  
  
`sh.shardCollection("testdb.order", {"_id": "hashed" })`  
  
插入1000条数据进行验证：  
  
```  
use testdb  
  
for (i = 1; i <= 1000; i=i+1){  
    db.order.insert({'price': 1})  
}  
  
```  
  
  
  
  
查看插入的数据量  
  
`db.order.find().count()`  
  
访问3个分片查看数据量  
  
```  
[root@mongodb01 ~]# mongo --host mongodb01 --port 27019  
  
shard1:PRIMARY> use testdb  
switched to db testdb  
shard1:PRIMARY> db.order.find().count()  
336  
  
[root@mongodb01 ~]# mongo --host mongodb01 --port 27020  
  
shard2:PRIMARY> use testdb  
switched to db testdb  
shard2:PRIMARY> db.order.find().count()  
326  
  
[root@mongodb01 ~]# mongo --host mongodb01 --port 27021  
  
shard3:PRIMARY> use testdb  
switched to db testdb  
shard3:PRIMARY> db.order.find().count()  
338  
  
```  

## 集群用户认证
证书生成
```
openssl rand -base64 756 > /data/mongodb/KeyFile.file
chmod 400 /data/mongodb/KeyFile.file
chown -R mongodb: /data/mongodb/KeyFile.file
```

配置文件
```
security:
  keyFile: /data/mongodb/KeyFile.file
  authorization: enabled
```

systemctl stop mongodb-config.service
systemctl stop mongodb-router.service
systemctl stop mongodb-shard1.service
systemctl stop mongodb-shard2.service 
systemctl stop mongodb-shard3.service 

systemctl start mongodb-config.service
systemctl start mongodb-router.service
systemctl start mongodb-shard1.service
systemctl start mongodb-shard2.service 
systemctl start mongodb-shard3.service 


systemctl status mongodb-config.service
systemctl status mongodb-router.service
systemctl status mongodb-shard1.service
systemctl status mongodb-shard2.service 
systemctl status mongodb-shard3.service 


https://www.cnblogs.com/pl-boke/p/10064489.html

