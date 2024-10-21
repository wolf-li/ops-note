# zookeeper 安装

### 文档适用于 3.4.10 和 3.7.0 版本其他版本请自行验证

普通用户安装

# 1.下载安装包，上传到服务器

下载地址：
<http://archive.apache.org/dist/zookeeper/zookeeper-3.4.11/zookeeper-3.4.11.tar.gz>

# 2.解压缩，移除安装包

```
sudo tar -xzf /data/zookeeper-3.4.11.tar.gz -C /data      
sudo mv /data/zookeeper-3.4.11 /data/zookeeper      
sudo rm -f /data/zookeeper-3.4.11.tar.gz      
```

# 3.更改配置文件

```
sudo cp /data/zookeeper/conf/zoo_sample.cfg /data/zookeeper/conf/zoo.cfg      
sudo sed -i "s:dataDir=/tmp/zookeeper:dataDir=/data/zookeeper/data:" /data/zookeeper/conf/zoo.cfg      
sudo chown -R 用户名：用户组 /data/zookeeper      
mkdir /data/zookeeper/data      
```

# 4.测试安装是否成功

/data/zookeeper/bin/zkServer.sh start
有下面显示，表示启动成功

```
ZooKeeper JMX enabled by default    
Using config: /data/zookeeper/bin/../conf/zoo.cfg    
Starting zookeeper ... STARTED    
```

使用本地客户端连接(使用默认配置进行连接 端口 2181，如果配置文件修改服务端端口，要使用 /data/zookeeper/bin/zkCli.sh -server IP:端口)
/data/zookeeper/bin/zkCli.sh
有下面显示且稳定，表示连接成功

```
WATCHER::    
    
WatchedEvent state:SyncConnected type:None path:null    
[zk: 127.0.0.1:2181(CONNECTED) 0]    
```

## zookeeper 配置文件说明

* tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
* initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 10*2000=20 秒
* syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 5*2000=10秒
* dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
* clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
* server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。
  
# 编写service 文件，使用 systemctl 控制应用  

创建 /etc/systemd/system/zk.service 文件  
填写下面内容  

```  
[Unit]  
Description=Zookeeper Daemon  
Documentation=http://zookeeper.apache.org  
Requires=network.target  
After=network.target  
  
[Service]  
Type=forking  
WorkingDirectory=/data/zookeeper  
User=用户名  
Group=用户组  
Environment=JAVA_HOME=/data/jdk1.8.0_301 
ExecStart=/data/zookeeper/bin/zkServer.sh start  
ExecStop=/data/zookeeper/bin/zkServer.sh stop  
ExecReload=/data/zookeeper/bin/zkServer.sh restart  
TimeoutSec=30  
Restart=on-failure  
  
[Install]  
WantedBy=default.target  
```  

systemctl daemon-reload
systemctl enable zk  
  
# 配置环境变量配置环境变量  

```  
echo "export ZK_HOME=/data/zookeeper" >> /etc/profile  
echo "export PATH=\$PATH:\$ZK_HOME/bin" >> /etc/profile  
source /etc/profile  
```  

zookeeper/bin 目录下的文件  
清理磁盘快照  
-rwxr-xr-x. 1 zk zk 1937 Nov  2  2017 zkCleanup.sh  
zookeeper 客户端  
-rwxr-xr-x. 1 zk zk 1534 Nov  2  2017 zkCli.sh  
设置 zookeeper 环境变量  
-rwxr-xr-x. 1 zk zk 2696 Nov  2  2017 zkEnv.sh  
zookeeper 服务端  
-rwxr-xr-x. 1 zk zk 6773 Nov  2  2017 zkServer.sh  
  
# zookeeper 集群启动  

1.在安装好 zookeeper 的基础上，修改配置文件  
在配置文件（zoo.cfg）中添加, 注意空格最好手写

```  
server.1=172.17.0.2:2888:3888
server.2=172.17.0.3:2888:3888
server.3=172.17.0.4:2888:3888
```  
    
2.创建 myid 文件  
在 配置文件中 dataDir 指定的目录下，创建 myid 文件。  
然后在该文件添加上一步 server 配置的对应 A 数字。  
后面的机器依次在相应目录创建myid文件，写上相应配置数字即可。  
3.验证  
zkServer.sh status  
查看集群节点状态。  
一台 leader 两台 follower  
  
### 添加防火墙策略

```bash
firewall-cmd --zone=public --add-port=2181/tcp --permanent 
firewall-cmd --zone=public --add-port=2888/tcp --permanent 
firewall-cmd --zone=public --add-port=3888/tcp --permanent 
firewall-cmd --reload
firewall-cmd --list-all
```

### 测试

启动客户端
./zkCli.sh -server localhost:2181
进行通信
ls /
新建文件
create /node node
在其他节点获取文件
get /node


### 报错

查看报错信息

1. 出现下面的情况，配置文件写的有问题
ERROR [main:QuorumPeerMain@99] - Invalid config, exiting abnormally
  
2. 启动失败 查看log 日志，多数为 配置问题，或 文件所属用户用户组


