## web client

SQLPad

### docker 部署方式
```
# The most minimal example, mapping port 3000 to local docker host
docker run -p 3000:3000 sqlpad/sqlpad:latest

# volume and env vars being set and run in background
# directory `~/docker-volumes` must be shared with docker to work
docker run --name sqlpad -p 13000:3000 --volume /data/mysql/sqlpad-mysql:/var/lib/sqlpad --detach sqlpad/sqlpad:latest

# 设置用户名密码，需要使用邮箱配置用户名
docker run --name sqlpad -p 13000:3000 --volume /data/mysql/sqlpad-mysql:/var/lib/sqlpad  -e  SQLPAD_ADMIN=admin@sqlpad.com   -e  SQLPAD_ADMIN_PASSWORD=admin --detach sqlpad/sqlpad:latest


# To list running docker images
docker ps

# To stop running docker image by name. (otherwise use container id from `docker ps`)
docker stop sqlpad
```

---
https://getsqlpad.com/en/getting-started/


