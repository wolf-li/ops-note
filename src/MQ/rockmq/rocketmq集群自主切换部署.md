# RocketMQ 集群自主切换模式部署

## 环境配置
* java 环境
decompression_dir=/data
tar xzf jdk-8u301-linux-x64.tar.gz -C ${decompression_dir}
echo "export JAVA_HOME=${decompression_dir}/jdk1.8.0_301
export JRE_HOME=${decompression_dir}/jdk1.8.0_301/jre
export CLASSPATH=.:\$JAVA_HOME/lib:\$JRE_HOME/lib:\$CLASSPATH
export PATH=\$JAVA_HOME/bin:\$JRE_HOME/bin:\$PATH" >> /etc/profile.d/jdk.sh
source /etc/profile.d/jdk.sh
java -version

* 添加用户
useradd -M rocketmq -s /sbin/nologin




---
官方文档：
https://rocketmq.apache.org/zh/docs/deploymentOperations/03autofailover

https://juejin.cn/post/7212597327579086908?searchId=20240130103747CF0980923EB92869609E