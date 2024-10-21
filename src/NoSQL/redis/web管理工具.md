# RedisInsight web 管理工具

## docker 部署
```shell
mkdir /data/redisinsight
chmod 777 /data/redisinsight
docker pull  redislabs/redisinsight:1.10.0  # 新版本有问题
docker run -it -d -v /data/redisinsight:/db -p 8001:8001 redislabs/redisinsight:1.10.0
```
