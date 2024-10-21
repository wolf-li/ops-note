# rocketmq 常用命令


## mqadmin 常用
```shell
# 查看当前所有 topic 的状态
mqadmin TopicList  -n $(hostname -I | awk '{print $1}'):9876 2>/dev/null | xargs -i sh -c "echo {} && mqadmin TopicStatus -n $(hostname -I | awk '{print $1}'):9876 -t {} 2>/dev/null "

# 获取topic的cluster
mqadmin topicClusterList -n $(hostname -I | awk '{print $1}'):9876 -t SCANRECORD

# 查看Topic列表信息
mqadmin topicList -n $(hostname -I | awk '{print $1}'):9876 2>/dev/null

# 查看topic 状态
mqadmin TopicStatus -n $(hostname -I | awk '{print $1}'):9876 -t TopicTest 2>/dev/null

# 查看集群状态
mqadmin clusterList -n $(hostname -I | awk '{print $1}'):9876 2>/dev/null

# 查看所有消费进度
mqadmin consumerProgress  -n $(hostname -I | awk '{print $1}'):9876 2>/dev/null

# 创建一个 topic
mqadmin updateTopic -n $(hostname -I | awk '{print $1}'):9876  -c DefaultCluster  -t test_rocketmq_4_9_0 2>/dev/null

# 查看 topic 路由
mqadmin topicRoute  -n $(hostname -I | awk '{print $1}'):9876 -t test_rocketmq_4_9_0 2>/dev/null

# 删除一个 topic
mqadmin deleteTopic  -c DefaultCluster  -n $(hostname -I | awk '{print $1}'):9876 -t test_rocketmq_4_9_0 2>/dev/null

# 查看所有消费组
mqadmin consumerProgress -n $(hostname -I | awk '{print $1}'):9876 2>/dev/null

# 查看指定消费组下的所有topic数据堆积情况
mqadmin consumerProgress -n $(hostname -I | awk '{print $1}'):9876 -g warning-group



# 查看某个 broker 具体状态
mqadmin brokerStatus -c rocketmq-cluster -n $(hostname -I | awk '{print $1}'):9876  -b $(hostname -I | awk '{print $1}'):10911

# broker 消费状态
mqadmin brokerConsumeStats -b  $(hostname -I | awk '{print $1}'):10911 -n  $(hostname -I | awk '{print $1}'):9876 

# 2表示只写权限，4表示只读权限，6表示读写权限
# 关闭 broker 写权限
mqadmin updateBrokerConfig -b $(hostname -I | awk '{print $1}'):10911 -n $(hostname -I | awk '{print $1}'):9876 -k brokerPermission -v 4

# 恢复 broker 权限
mqadmin updateBrokerConfig -b $(hostname -I | awk '{print $1}'):10911 -n $(hostname -I | awk '{print $1}'):9876 -k brokerPermission -v 6

# 查看 broker 配置
mqadmin getBrokerConfig -b  $(hostname -I | awk '{print $1}'):10911 -n  $(hostname -I | awk '{print $1}'):9876 

```
## mqadmin 命令详解
整体来说，命令分为如下几大类：

### 1. 集群相关：

clusterList：查看集群列表 
clusterRT：测试集群的响应耗时

### 2. broker相关  

updateBrokerConfig ：更新broker的配置
brokerStatus ：获取broker的运行时状态数据
wipeWritePerm：设置某broker为只读
getBrokerConfig：获取broker的配置信息
### 3.topic相关：

updateTopic：创建或更新topic
deleteTopic ：从nameserver和broker中删除topic信息
updateTopic：更新topic的perm信息
topicRoute：获取topic的路由信息
topicStatus： 获取topic的当前状态信息
topicClusterList：获取topic对应的集群信息
topicList：查看集群中的topic列表
statsAll: 查看所有topic已经对应的consumer的消费进度
cleanUnusedTopic：清理集群中无用的topic

### 4.message相关：
queryMsgById: 通过message的id查询消息
queryMsgByKey：通过message的key查询消息
queryMsgByUniqueKey：通过message的UniqueKey查询消息
queryMsgByOffset：通过偏移查询message 、
printMsg：打印某条message的详情
printMsgByQueue：打印某个queue（队列）里的消息的详情
sendMsgStatus：向broker发送消息
sendMessage：发送一条消息
producerConnection：查询producer的信息(socket连接，客户端版本） 

### 5. 消费相关:
consumerConnection ：查询consumer的信息（socker连接，客户端版本，消费组）
consumerProgress ：查询consumer的消费进度，tps
consumerStatus ：查询consumer的的内部数
brokerConsumeStats：查看所有topic对应的消费数据（broker的offset，consumer的offset，是否有diff）  
cleanExpiredCQ：清理集群中无用的消费组cloneGroupOffset clone offset from other group.
cloneGroupOffset ：克隆指定topic下某个消费组的消费进度到指定消费组
resetOffsetByTime：设置consumer的offset到某个时间点
checkMsgSendRT：测试消费发生的响应耗时
consumeMessage：消费消息
queryCq：查询consumer指定队列和索引位置的消费信息
updateSubGroup：创建或者更新消费组（订阅组）
deleteSubGroup：从broker中删除消费组（订阅组）

### 6. nameserver相关:
updateKvConfig：更新nameserver的kv配置
deleteKvConfig：删除nameserver的kv配置      
getNamesrvConfig：获取nameserver的配置
updateNamesrvConfig：更新nameserver的配置

### 7. acl相关：
updateAclConfig：更新acl配置   
deleteAccessConfig：删除acl配置的账户  
clusterAclConfigVersion：查看集群的acl配置版本
updateGlobalWhiteAddr：更新acl里面的白名单 
getAccessConfigSubCommand：查看acl的配置信息

### 8. 其他：
updateOrderConf Create or update or delete order conf
startMonitoring Start Monitoring
allocateMQ Allocate MQ

