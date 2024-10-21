# kafka 初探
## 简介
kafka 是分布式流平台
#### 主要功能
* 发布和订阅记录流，类似于消息队列或企业消息传递系统。
* 以容错持久的方式存储记录流。
* 在记录流发生时进行处理。
#### Kafka通常用于两大类应用:
构建实时流数据管道，在系统或应用程序之间可靠地获取数据
构建实时流应用程序，转换或响应数据流
#### 基本概念
* kafka 作为一个或多个集群来运行，可以在不同数据中心中进行迁移
* kafka 存储数据流叫做 topics
* 每个记录都有 key value timestamp
#### kafka 有四个核心 api
* Producer API 允许应用程序发布一个或多个Kafka topic 的记录流。
* Consumer API 允许应用程序订阅一个或多个主题，并处理给它们产生的记录流。
* Streams API 允许应用程序充当流处理器，从一个或多个主题消费输入流，并产生一个输出流到一个或多个输出主题，有效地将输入流转换为输出流。
* Connector API 允许构建和运行可重用的生产者或消费者，将Kafka主题连接到现有的应用程序或数据系统。例如，关系数据库的连接器可能捕获对表的每个更改。

## 参考文献：
[kafka 2.1 Doc](https://kafka.apache.org/21/documentation.html)


# java 环境安装
## 下载安装包
安装包 url  https://repo.huaweicloud.com/java/jdk/11.0.2+9/jdk-11.0.2_linux-x64_bin.tar.gz

## 解压缩文件
`tar xzf jdk1.8.0_301.tar.gz`

## 配置环境变量
```
 tar xzf /home/app/jdk-8u301-linux-x64.tar.gz -C /data
echo "export JAVA_HOME=/data/jdk1.8.0_301
PATH=\$PATH:\$JAVA_HOME/bin
export JAVA_HOME PATH" >> /etc/profile
source /etc/profile
java -version
```
## 验证jdk

```
java -version
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
```
### 注意zookeeper 是域名还是ip之间进行配置

tar xzf /home/app/kafka_2.12-2.8.0.tgz -C /data
cd /data
mv /data/kafka_2.12-2.8.0 /data/kafka
mkdir /data/kafka/logs
cat <<EOF > /etc/hosts
10.250.64.33 zookeeper1
10.250.64.34 zookeeper2
10.250.64.35 zookeeper3
EOF

修改配置文件 /data/kafka/config/server.properties
```
broker.id=0   #另外两个修改为1，2
listeners=PLAINTEXT://192.168.23.161:9092   #对应本机ip
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/tmp/kafka-logs
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=192.168.23.161:2181,192.168.23.162:2181,192.168.23.163:2181
delete.topic.enble=true
zookeeper.connection.timeout.ms=18000
group.initial.rebalance.delay.ms=0
```



启动 kafka
/data/kafka/bin/kafka-server-start.sh  -daemon /data/kafka/config/server.properties

测试
## 创建 topic
/data/kafka/bin/kafka-topics.sh --create --zookeeper zookeeper1:2181 --replication-factor 3 --partitions 2 --topic google
## 查看 topic
/data/kafka/bin/kafka-topics.sh --list --zookeeper zookeeper2:2181
## 在创建好的 topic 生产消息
/data/kafka/bin/kafka-console-producer.sh --broker-list 10.250.64.40:9092 --topic google 
/data/kafka/bin/kafka-console-producer.sh --bootstrap-server 10.250.64.40:9092 --topic google

/data/kafka/bin/kafka-console-consumer.sh --bootstrap-server 10.250.64.40:9092 --topic google

/data/kafka/bin/kafka-topics.sh --delete --zookeeper zookeeper3:2181 --topic google

jps -ml


### 安装报错处理
1. 报错如下   “#å¦å¤ä¸¤ä¸ªä¿®æ¹ä¸º1ï¼2” 非法字符，配置文件中不要出现中文及时加注释
```
[2022-03-04 10:19:04,936] INFO Registered kafka:type=kafka.Log4jController MBean (kafka.utils.Log4jControllerRegistration$)
[2022-03-04 10:19:05,340] ERROR Exiting Kafka due to fatal exception (kafka.Kafka$)
org.apache.kafka.common.config.ConfigException: Invalid value 2   #å¦å¤ä¸¤ä¸ªä¿®æ¹ä¸º1ï¼2 for configuration broker.id: Not a number of type INT
	at org.apache.kafka.common.config.ConfigDef.parseType(ConfigDef.java:742)
	at org.apache.kafka.common.config.ConfigDef.parseValue(ConfigDef.java:490)
	at org.apache.kafka.common.config.ConfigDef.parse(ConfigDef.java:483)
	at org.apache.kafka.common.config.AbstractConfig.<init>(AbstractConfig.java:108)
	at org.apache.kafka.common.config.AbstractConfig.<init>(AbstractConfig.java:142)
	at kafka.server.KafkaConfig.<init>(KafkaConfig.scala:1386)
	at kafka.server.KafkaConfig.<init>(KafkaConfig.scala:1389)
	at kafka.server.KafkaConfig$.fromProps(KafkaConfig.scala:1327)
	at kafka.Kafka$.buildServer(Kafka.scala:67)
	at kafka.Kafka$.main(Kafka.scala:87)
	at kafka.Kafka.main(Kafka.scala)
[app@i-ZxlpPmouy-8 kafka]$ vi config/server.properties
```
解决方法：
删除非英文字符

命令工具|作用
--|--
kafka-broker-api-versions.sh | This tool helps to retrieve broker version information.
其他工具可以参考 https://docs.confluent.io/kafka/operations-tools/kafka-tools.html#search-by-tool-name


kafka 内部 topic
__consumer_offsets: Every consumer group maintains its offset per topic partitions. Since v0.9 the information of committed offsets for every consumer group is stored in this internal topic (prior to v0.9 this information was stored on Zookeeper). When the offset manager receives an OffsetCommitRequest, it appends the request to a special compacted Kafka topic named __consumer_offsets. Finally, the offset manager will send a successful offset commit response to the consumer, only when all the replicas of the offsets topic receive the offsets.

_schemas: This is an internal topic used by the Schema Registry which is a distributed storage layer for Avro schemas. All the information which is relevant to schema, subject (with its corresponding version), metadata and compatibility configuration is appended to this topic. The schema registry in turn, produces (e.g. when a new schema is registered under a subject) and consumes data from this topic.

---
kafka 相关文章：https://mp.weixin.qq.com/s/7F2dfDBia2--dzDRyccgbQ
https://mp.weixin.qq.com/s/ah-q06AAANJJ_rOgHAcc6Q
https://mp.weixin.qq.com/s/M9M-ib3V_5XcqQisE7YUrw
https://mp.weixin.qq.com/s/G2dMxG42RnHi4ZWNkQRgBQ
https://docs.confluent.io/kafka/operations-tools/kafka-tools.html#search-by-tool-name
https://blog.51cto.com/u_15290941/5294373
