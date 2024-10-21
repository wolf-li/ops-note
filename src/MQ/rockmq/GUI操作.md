# rocketmq Dashboard

适合高版本，低版本使用有问题
```bash
docker pull apacherocketmq/rocketmq-dashboard:latest
```

# rocketmq console
```bash
docker pull styletang/rocketmq-console-ng
docker run -d --name rocketmq-dashboard -e "JAVA_OPTS=-Drocketmq.namesrv.addr=10.240.53.47:9876" -p 8080:8080 -t  styletang/rocketmq-console-ng:latest
```

