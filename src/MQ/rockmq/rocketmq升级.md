# 4.3.1 升级 4.9.0
## 升级过程
1. 添加 broker 


新版本与旧版本可以共用数据

查看 rocketmq 消费情况
```shel
mqadmin consumerProgress  -n $(hostname -I | awk '{print $1}'):9876 2>/dev/null
```
broker.conf 配置文件

> brokerClusterName = DefaultCluster
> brokerName = broker-a  # 需要修改名字
> brokerId = 0  # 0代表 master 1 代表 slave
> deleteWhen = 04
> fileReservedTime = 48
> brokerRole = ASYNC_MASTER
> flushDiskType = ASYNC_FLUSH
> namesrvAddr = 10.240.53.94:9876;10.240.53.94:9877
> brokerIP1 = 10.240.53.94
> listenPort=11911   # 需要修改
> storePathRootDir = /data/rocketmq-4.9.3/store
> storePathCommitLog = /data/rocketmq-4.9.3/store/commitlog
> storePathConsumeQueue = /data/rocketmq-4.9.3/store/consumequeue
> storePathIndex = /data/rocketmq-4.9.3/store/index
> storeCheckpoint = /data/rocketmq-4.9.3/store/checkpoint
> abortFile = /data/rocketmq-4.9.3/store/abort
> maxMessageSize = 65536

nameserver 需要修改 systemd 配置文件
```
ExecStart=/data/rocketmq-4.9.3/bin/mqnamesrv -c /data/rocketmq-4.9.3/conf/namesrv.properties
```

/data/rocketmq-4.9.3/conf/namesrv.properties 文件内容
>listenPort=9877


调节 jvm


```
unzip -d /data/ rocketmq-all-4.9.0-bin-release.zip
mkdir /data/rocketmq-all-4.9.0-bin-release/{data,logs}
cp -R /data/rocketmq/data/*  /data/rocketmq-all-4.9.0-bin-release/data/
chown -R rocketmq:rocketmq /data/rocketmq-all-4.9.0-bin-release/

cat << EOF > /data/rocketmq-all-4.9.0-bin-release/conf/broker.conf 
brokerClusterName = DefaultCluster
brokerName = broker-b
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
namesrvAddr=$(hostname -I | awk '{print $1}'):9876
brokerIP1=$(hostname -I | awk '{print $1}')
# listenPort=11911
storePathRootDir=/data/rocketmq-all-4.9.0-bin-release/data
storePathCommitLog=/data/rocketmq-all-4.9.0-bin-release/logs
EOF

# 调整 jvm 文件名称（runbroker.sh）
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m"

cat > /etc/systemd/system/rmq_broker-4.9.service << EOF
[Unit]
# 服务描述
Description=rmq_broker
Requires=network.service
After=network.target

[Service]
User=rocketmq
Group=rocketmq
StartLimitBurst=2
StartLimitInterval=30
Environment=JAVA_HOME=/data/jdk1.8.0_301
ExecStart=/data/rocketmq-all-4.9.0-bin-release/bin/mqbroker -c /data/rocketmq-all-4.9.0-bin-release/conf/broker.conf
ExecStop=/data/rocketmq-all-4.9.0-bin-release/bin/mqshutdown broker 
KillMode=none
Restart=on-failure
RestartSec=30s

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl start rmq_broker-4.9.service




systemctl stop rmq_broker.service; systemctl start rmq_broker-4.9.service 


nameserver

cat > /etc/systemd/system/rmq_namesrv-4.9.service << EOF
[Unit]
# 服务描述
Description=rmq_namesrv
Requires=network.service
After=network.target

[Service]
User=rocketmq
Group=rocketmq
StartLimitBurst=2
StartLimitInterval=30
Environment=JAVA_HOME=/data/jdk1.8.0_301
ExecStart=/data/rocketmq-all-4.9.0-bin-release/bin/mqnamesrv
ExecStop=/data/rocketmq-all-4.9.0-bin-release/bin/mqshutdown namesrv
KillMode=none
Restart=on-failure
# time to sleep before restarting the service
RestartSec=30s

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl stop rmq_namesrv.service; systemctl start rmq_namesrv-4.9.service
```


```
broker的写权限关闭
bin/mqadmin updateBrokerConfig -b 192.168.x.x:10911 -n 192.168.x.x:9876 -k brokerPermission -v 4
 
恢复该节点的写权限
bin/mqadmin updateBrokerConfig -b 192.168.x.x:10911 -n 192.168.x.x:9876 -k brokerPermission -v 6
```


---
参考文档：
