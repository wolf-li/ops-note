# rocketmq 集群部署（同步双写）
### rocketmq 需要环境 jdk
安装 jdk 1.8
#### 1. 上传 jdk 安装包
#### 2. 解压缩安装包
` tar xzf /home/app/jdk-8u301-linux-x64.tar.gz -C /data`

#### 3. 配置环境变量
```
echo "JAVA_HOME=/data/jdk1.8.0_301
PATH=\$PATH:\$JAVA_HOME/bin
export JAVA_HOME PATH" >> /etc/profile
source /etc/profile
```
#### 4. 验证jdk

```
java -version
java version "11.0.2" 2019-01-15 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.2+9-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.2+9-LTS, mixed mode)
```
#### 5. 单机启动
```
  cd /data/rocketmq
  nohup sh bin/mqnamesrv &
  nohup sh bin/mqbroker -n 10.250.76.96:9876 &
```

### rocketmq 集群安装
服务器
|IP|用途|
--|--
10.250.65.2|NameServer-1、broker-a-master
10.250.65.3|NameServer-2、broker-a-slave 
10.250.65.4|broker-b-master 
10.250.65.5|broker-b-slave

cd /data
unzip /home/app/rocketmq-all-4.3.1-bin-release.zip
mv rocketmq-all-4.3.1-bin-release rocketmq
mkdir -p /data/rocketmq/store{commitlog,consumequeue,index,checkpoint,abort}
mkdir -p /data/rocketmq/logs
touch /data/rocketmq/logs/mq.log
useradd rocketmq -s /sbin/nologin
chown -R rocketmq:rocketmq rocketmq/


### 启动 nameserver


配置 systemd 文件
/etc/systemd/system/rmq_namesrv.service
```
[Unit]
Description=rmq_namesrv
Requires=network.service
After=network.target

[Service]
User=rocketmq
Group=rocketmq
StartLimitBurst=2
StartLimitInterval=30
Environment=JAVA_HOME=/data/jdk1.8.0_301
ExecStart=/data/rocketmq/bin/mqnamesrv
ExecStop=/data/rocketmq/bin/mqshutdown namesrv
KillMode=none
Restart=on-failure
RestartSec=30s

[Install]
WantedBy=multi-user.target
```


### 修改配置文件
#### 运行配置文件(可选)
```
bin/runserver.sh
bin/runbroker.sh
bin/tool.sh
```

### broker 配置文件 
#### broker-a.properties
```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master，1 表示 Slave
brokerId=0
# broker 当前 ip
brokerIP1 = 10.240.58.28
#nameServer地址，分号分割
namesrvAddr=192.168.30.208:9876;192.168.30.209:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
# 新 4.9 版本需要使用下面的参数
# mappedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
#storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
#storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
#storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
#storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
#storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
#abortFile=/data/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
 
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
 
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```
###### systemd 配置文件
/etc/systemd/system/rmq_broker-a.properties.service
```
[Unit]
Description=rmq_namesrv
Requires=network.service
After=network.target

[Service]
User=rocketmq
Group=rocketmq
StartLimitBurst=2
StartLimitInterval=30
Environment=JAVA_HOME=/data/jdk1.8.0_301
ExecStart=/data/rocketmq/bin/mqbroker -c /data/rocketmq/conf/2m-2s-sync/broker-a.properties
ExecStop=/data/rocketmq/bin/mqshutdown broker
KillMode=none
Restart=on-failure
RestartSec=30s

[Install]
WantedBy=multi-user.target
```

#### broker-a-s.properties
```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master，1 表示 Slave
brokerId=1
#nameServer地址，分号分割
# broker 当前 ip
brokerIP1 = 10.240.58.28
namesrvAddr=192.168.30.208:9876;192.168.30.209:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=false
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=false
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
 
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
 
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```
##### 配置 systemd 文件
/etc/systemd/system/rmq_broker-a-s.properties.service
```
[Unit]
Description=rmq_namesrv
Requires=network.service
After=network.target

[Service]
User=rocketmq
Group=rocketmq
StartLimitBurst=2
StartLimitInterval=30
Environment=JAVA_HOME=/data/jdk1.8.0_301
ExecStart=/data/rocketmq/bin/mqbroker -c /data/rocketmq/conf/2m-2s-sync/broker-a-s.properties
ExecStop=/data/rocketmq/bin/mqshutdown broker
KillMode=none
Restart=on-failure
RestartSec=30s

[Install]
WantedBy=multi-user.target
```

#### broker-b.properties
```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master，1 表示 Slave
brokerId=0
#nameServer地址，分号分割
# broker 当前 ip
brokerIP1 = 10.240.58.28
namesrvAddr=192.168.30.208:9876;192.168.30.209:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
 
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
 
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

##### 配置 systemd 文件
/etc/systemd/system/rmq_broker-b.properties.service
```
[Unit]
Description=rmq_namesrv
Requires=network.service
After=network.target

[Service]
User=rocketmq
Group=rocketmq
StartLimitBurst=2
StartLimitInterval=30
Environment=JAVA_HOME=/data/jdk1.8.0_301
ExecStart=/data/rocketmq/bin/mqbroker -c /data/rocketmq/conf/2m-2s-sync/broker-b.properties
ExecStop=/data/rocketmq/bin/mqshutdown broker
KillMode=none
Restart=on-failure
RestartSec=30s

[Install]
WantedBy=multi-user.target
```

#### broker-b-s.properties
```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master，1 表示 Slave
brokerId=1
#nameServer地址，分号分割
# broker 当前 ip
brokerIP1 = 10.240.58.28
namesrvAddr=192.168.30.208:9876;192.168.30.209:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
 
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
 
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

##### 配置 systemd 文件
/etc/systemd/system/rmq_broker-b-s.properties.service
```
[Unit]
Description=rmq_namesrv
Requires=network.service
After=network.target

[Service]
User=rocketmq
Group=rocketmq
StartLimitBurst=2
StartLimitInterval=30
Environment=JAVA_HOME=/data/jdk1.8.0_301
ExecStart=/data/rocketmq/bin/mqbroker -c /data/rocketmq/conf/2m-2s-sync/broker-b-s.properties
ExecStop=/data/rocketmq/bin/mqshutdown broker
KillMode=none
Restart=on-failure
RestartSec=30s

[Install]
WantedBy=multi-user.target
```




查看是否存在 mq 进程
ps aux|grep java|grep mq

#### 查看集群
sh bin/mqadmin clusterList -n 10.250.65.3:9876
结果如下
```
#Cluster Name     #Broker Name            #BID  #Addr                  #Version                #InTPS(LOAD)       #OutTPS(LOAD) #PCWait(ms) #Hour #SPACE
rocketmq-cluster  broker-a                0     10.240.58.28:10911     V4_3_1                   0.00(0,0ms)         0.00(0,0ms)          0 457594.47 -1.0000
rocketmq-cluster  broker-a                1     10.240.58.29:10911     V4_3_1                   0.00(0,0ms)         0.00(0,0ms)          0 457594.47 0.0025
rocketmq-cluster  broker-b                0     10.240.58.30:10911     V4_3_1                   0.00(0,0ms)         0.00(0,0ms)          0 457594.47 -1.0000
rocketmq-cluster  broker-b                1     10.240.58.31:10911     V4_3_1                   0.00(0,0ms)         0.00(0,0ms)          0 457594.47 0.0009
```

### 备注
5.1.2 版本需要修改 bin/runserver.sh 删除部分代码，否则 systemd 无法启动
```bash
find_java_home()
{
    case "`uname`" in
        Darwin)
          if [ -n "$JAVA_HOME" ]; then
            JAVA_HOME=$JAVA_HOME
            return
          fi
            JAVA_HOME=$(/usr/libexec/java_home)
        ;;
        *)
            JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac))))
        ;;
    esac
}

find_java_home
```


--- 
参考文档：
https://github.com/apache/rocketmq/blob/master/docs/cn/operation.md
https://www.cnblogs.com/lwhctv/p/12289802.html
