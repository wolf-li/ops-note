## MQ 简介
MQ MessageQueue 消息队列，队列是一种FIFO先进先出的数据结构，消息由生产这发送到 MQ 进行排队，然后按照原来的顺序交由消息的消费者进行处理 QQ 和微信就是典型的 MQ

MQ 的左右主要有以下方面
* 异步
* 解耦
    服务之间进行解耦，可以减少服务之间的影响，提高系统的稳定性以及可扩展性
    实现数据分发，生产者发送一个消息后，可以由一个或者多个消费者进行消费，并且消费的增加或减少对生产者没有影响
* 削峰
    以稳定的系统资源应对突发的流量冲击

### MQ的缺点
* 系统可用性降低
    系统引入的外部依赖增多，系统的稳定性就会变差，一旦 MQ 宕机，会对业务产生影响，这就要考虑如何保证 MQ 的高可用
* 系统复杂度提高
    引入 MQ 对系统的复杂度会大大提高，以前服务之间可以进行同步的服务调用，引入后，数据链路就会变得复杂，并且会带来其他一些问题，比如：如何保证消费不会丢失？不会被重复调用？怎么保证消息的顺序性等问题。
* 消息一致性问题


配置环境变量

源码编译安装
```
# 1. git clone 源码
git clone https://github.com/apache/rocketmq.git
# 2. 切换到指定版本
git checkout 
```

### broker 使用端口
用途 | 使用端口
--|--
listenPort| 10911
fastListenPort| 10909
haListenPort| 10912

### 修改日志保存位置
#### 方法一 修改配置文件，数量过多
rocketmq-5.1.2/conf/rmq.broker.logback.xml
rocketmq-5.1.2/conf/rmq.client.logback.xml
rocketmq-5.1.2/conf/rmq.controller.logback.xml
rocketmq-5.1.2/conf/rmq.namesrv.logback.xml
rocketmq-5.1.2/conf/rmq.proxy.logback.xml
rocketmq-5.1.2/conf/rmq.tools.logback.xml

#### 方法二 修改变量
一共需要修改两个文件
bin/runnamesrv.sh和bin/runserver.sh
都是在
```
JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${JAVA_HOME}/jre/lib/ext:${BASE_DIR}/lib:${JAVA_HOME}/lib/ext"
```
后面添加
```
#rewrite user home 
JAVA_OPT="${JAVA_OPT} -Duser.home=新目录"
```
重启namesrv,broker即可


https://segmentfault.com/a/1190000040395617

## 问题
5.1.2 runserver.sh runbroker.sh find_java_home 函数有问题注释


文档连接
[RocketMQ Console](https://rocketmq-1.gitbook.io/rocketmq-connector/rocketmq-connect/rocketmq-console/kong-zhi-tai-jian-jie)
[Rocketmq doc](https://rocketmq.apache.org/docs/quick-start/)
[broker 使用端口](https://blog.csdn.net/changqing5818/article/details/113973803)
