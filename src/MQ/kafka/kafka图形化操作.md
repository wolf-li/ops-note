# Kafdrop docker 部署
```bash
docker run -d --rm -p 9000:9000 \
    -e KAFKA_BROKERCONNECT=192.168.100.153:9092 \
    -e JVM_OPTS="-Xms32M -Xmx64M" \
    -e SERVER_SERVLET_CONTEXTPATH="/" \
    obsidiandynamics/kafdrop
```




# cmak docker 部署

仅支持 kafka 2.4
docker run -p 9000:9000 -e ZK_HOSTS="192.168.58.132:2181" --name docker_cmak -d vimagick/cmak


---
参考连接：
https://blog.csdn.net/x763795151/article/details/121685576