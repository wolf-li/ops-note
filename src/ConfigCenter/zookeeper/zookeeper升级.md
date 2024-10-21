# zookeeper 集群升级  
升级需求  
版本升级：zookeeper集群节点数量和拓扑结构不变，只改变zookeeper的版本  
  
集群架构：  
|节点|ip|  
|--|--|  
|leader|10.250.60.33|  
|follower1|10.250.60.32|  
|follower2|19.250.60.34|  
  
  
旧版本 3.4.6  
新版本 3.7.0  
注意事项：  
1. zookeeper 新版本对 jdk 有要求  
文档提示：  
If you are going to compile with Java 1.8, you should use a  
recent release at u211 or above.  
2. zookeeper 软件包下载  
`apache-zookeeper-3.7.0-bin.tar.gz`  
`apache-zookeeper-3.7.0.tar.gz`  
选择 apache-zookeeper-3.7.0-bin.tar.gz 里面是打包好的二进制文件可以在安装好 jdk 环境后，启动。而另一个不能，需要打包。  
  
## 升级部署：  
### 1. 环境准备  
查看jdk版本是否符合要求，没有要更新，上传新的 zookeeper 包  
sudo tar xzf apache-zookeeper-3.7.0-bin.tar.gz -C /data  
sudo rm -f apache-zookeeper-3.7.0-bin.tar.gz  
  
### 2. 修改配置测试  
配置内容跟之前zookeeper集群有所不同，不过我们利用的是zookeeper选举同步这个特点来进行逐步替换，并且为了保证集群服务的最小影响，我们需要从从节点开始替换！-----------当前33是leader  
先将32节点旧版本的zookeeper服务停掉  
`/data/zookeeper-3.4.6/bin/zkServer.sh stop`

在新版本内创建目录，myid 文件  
对应旧版本配置文件  
dataDir=/data/zookeeper-3.4.6/zkData  
dataLogDir=/data/zookeeper-3.4.6/logs  
echo "1" > /data/zookeeper-3.4.6/zkData/myid  
server.1=10.250.60.32:2888:3888  
  
旧版本配置文件  
```  
# The number of milliseconds of each tick  
tickTime=2000  
# The number of ticks that the initial  
# synchronization phase can take  
initLimit=10  
# The number of ticks that can pass between  
# sending a request and getting an acknowledgement  
syncLimit=5  
# the directory where the snapshot is stored.  
# do not use /tmp for storage, /tmp here is just  
# example sakes.  
dataDir=/data/zookeeper-3.4.6/zkData  
dataLogDir=/data/zookeeper-3.4.6/logs  
# the port at which the clients will connect  
clientPort=2181  
server.1=10.250.60.32:2888:3888  
server.2=10.250.60.33:2888:3888  
server.3=10.250.60.34:2888:3888  
authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider  
authProvider.2=org.apache.zookeeper.server.auth.SASLAuthenticationProvider  #开启认证功能  
authProvider.3=org.apache.zookeeper.server.auth.SASLAuthenticationProvider  #开启认证功能  
requireClientAuthScheme=sasl  
jaasLoginRenew=3600000  
  
# the maximum number of client connections.  
# increase this if you need to handle more clients  
#maxClientCnxns=60  
#  
# Be sure to read the maintenance section of the  
# administrator guide before turning on autopurge.  
#  
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance  
#  
# The number of snapshots to retain in dataDir  
#autopurge.snapRetainCount=3  
# Purge task interval in hours  
# Set to "0" to disable auto purge feature  
#autopurge.purgeInterval=1  
```  
  
新版本配置文件  
```  
# The number of milliseconds of each tick  
tickTime=2000  
# The number of ticks that the initial  
# synchronization phase can take  
initLimit=10  
# The number of ticks that can pass between  
# sending a request and getting an acknowledgement  
syncLimit=5  
# the directory where the snapshot is stored.  
# do not use /tmp for storage, /tmp here is just  
# example sakes.  
dataDir=/data/apache-zookeeper-3.7.0-bin/zkData  
dataLogDir=/data/apache-zookeeper-3.7.0-bin/logs  
# the port at which the clients will connect  
clientPort=2181  
server.1=10.250.60.32:2888:3888  
server.2=10.250.60.33:2888:3888  
server.3=10.250.60.34:2888:3888  
```  
### 3. 启动 zk ，查看是否启动  
`/data/apache-zookeeper-3.7.0-bin/bin/zkServer.sh start`  
`/data/apache-zookeeper-3.7.0-bin/bin/zkServer.sh status`  
`/data/apache-zookeeper-3.7.0-bin/bin/zkCli.sh `   
启动后你会发现通过客户端发现，它拥有了跟33，34节点同样的数据，说明数据热同步成功！  
其他节点照此操作，最后操作 leader 节点。  
总结：如果忽略选举时间，基本上已经完全实现了热替换，数据也能够同步过来！前提是zookeeper版本支持向后兼容，一般都支持滴  
  
  