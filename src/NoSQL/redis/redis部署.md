
第一步：下载Redis、安装gcc环境
https://download.redis.io/releases/

yum -y install gcc



第二步：解压并编译Redis压缩包
[root@node2 data]# tar -xf redis-3.2.8.tar.gz

[root@node2 data]# pwd

/data

编译Redis：

[root@node2 data]# cd redis-3.2.8/

[root@node2 redis-3.2.8]# pwd

/data/redis-3.2.8

[root@node2 redis-3.2.8]# make MALLOC=libc



第三步：创建相关目录
根据端口名，创建目录

mkdir -p /data/redis-3.2.8-cluster/7000

mkdir -p /data/redis-3.2.8-cluster/7001

mkdir -p /data/redis-3.2.8-cluster/7002

mkdir -p /data/redis-3.2.8-cluster/7003

mkdir -p /data/redis-3.2.8-cluster/7004

mkdir -p /data/redis-3.2.8-cluster/7005



第四步：修改Redis配置文件
[root@node2 data]# cd redis-3.2.8/

[root@node2 redis-3.2.8]# vim redis.conf

修改Redis配置文件的参数

bind 127.0.0.1 #修改为主机ip

daemonize yes #开启守护进程

appendonly yes #开启Redis持久化

cluster-enabled yes    #开启集群模式

cluster-config-file nodes.conf

cluster-node-timeout 5000

port 7000

pidfile /data/redis-3.2.8-cluster/7000/redis_7000.pid

logfile "/data/redis-3.2.8-cluster/7000/7000.log"

dir /data/redis-3.2.8-cluster/7000/

Redis集群配置文件样例

bind 10.250.61.34
protected-mode yes
port 7000
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile "/data/redis-3.2.8-cluster/7000/redis_7000.pid"
loglevel notice
logfile "/data/redis-3.2.8-cluster/7000/7000.log"
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump.rdb"
dir "/data/redis-3.2.8-cluster/7000"
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
cluster-enabled yes
cluster-config-file "nodes.conf"
cluster-node-timeout 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes


第五步：将配置文件cp新建的端口目录下，并修改配置文件相关路径和端口
[root@node2 redis-3.2.8]# cp ./redis.conf  /data/redis-3.2.8-cluster/7000

[root@node2 redis-3.2.8]# cp ./redis.conf  /data/redis-3.2.8-cluster/7001

[root@node2 redis-3.2.8]# cp ./redis.conf  /data/redis-3.2.8-cluster/7002

[root@node2 redis-3.2.8]# cp ./redis.conf  /data/redis-3.2.8-cluster/7003

[root@node2 redis-3.2.8]# cp ./redis.conf  /data/redis-3.2.8-cluster/7004

[root@node2 redis-3.2.8]# cp ./redis.conf  /data/redis-3.2.8-cluster/7005

修改配置文件（根据实际情况修改对应目录）：

第六步：启动Redis进程
cd /data/redis-3.2.8-cluster/7000

/data/redis-3.2.8/src/redis-server ./redis.conf



cd /data/redis-3.2.8-cluster/7001

/data/redis-3.2.8/src/redis-server ./redis.conf



cd /data/redis-3.2.8-cluster/7002

/data/redis-3.2.8/src/redis-server ./redis.conf



cd /data/redis-3.2.8-cluster/7003

/data/redis-3.2.8/src/redis-server ./redis.conf



cd /data/redis-3.2.8-cluster/7004

/data/redis-3.2.8/src/redis-server ./redis.conf



cd /data/redis-3.2.8-cluster/7005

/data/redis-3.2.8/src/redis-server ./redis.conf



[root@node2 redis-3.2.8]# ps aux|grep redis    #查看Redis是否正常启动

  

第七步：安装ruby环境（2.2.0以上）
1.yum 安装

           yum -y install ruby

           yum -y install rubygems  # ruby包的管理器,用来下载ruby的包



2.离线安装

下载地址：

http://www.ruby-lang.org/en/downloads/releases/

编译安装ruby：

[root@node2 redis-3.2.8]# tar -xf ruby-2.2.3.tar.gz   

[root@node2 redis-3.2.8]# cd ruby-2.2.3

[root@node2 redis-3.2.8]# ./configure

[root@node2 redis-3.2.8]# make;make install



安装rubygems

下载地址：

https://rubygems.org/gems/redis/versions

gem install -l redis-2.2.2.gem



第八步：创建集群（ip更具实际主机ip修改）
[root@node2 redis-3.2.8]# /data/redis-3.2.8/src/redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005





第九步：登录Redis验证
/data/redis-3.2.8/src/redis-cli -p 7000 -h 127.0.0.1  -c   # -c 登录集群需要加上-c选项

> set test 111



/data/redis-3.2.8/src/redis-cli -p 7001 -h 127.0.0.1  -c  #登录其他端口查看是否有数据

> get test



cluster nodes 查询集群结点信息；

cluster info 查询集群状态信。



遇到问题：



make MALLOC=libc   #编译报错
cc: error: ../deps/hiredis/libhiredis.a: No such file or directory

cc: error: ../deps/lua/src/liblua.a: No such file or directory

make[1]: *** [redis-server] Error 1

make[1]: Leaving directory `/usr/local/src/redis-4.0.1/src'



解决方法：

cd /data/redis-3.2.8/deps

make geohash-int hiredis jemalloc linenoise lua

Cd ..

make MALLOC=libc

设置Redis登录密码
./data/redis-3.2.8/src/redis-cli -p 7000 -h 127.0.0.1  -c 

#登录每个端口的Redis执行下面的命令

config set masterauth ..pansoft20210418
config set requirepass ..pansoft20210418
auth ..pansoft20210418
config rewrite



Redis集群相关操作


1.Redis集群新增Redis节点（新增Redis节点配置文件开启集群模式）
    启动新的Redis节点后，通过redis-trib.rb 进程进行添加节点

        ./redis-trib.rb add-node 127.0.0.1:6385 127.0.0.1:6384

           127.0.0.1:6385 为新增节点

           127.0.0.1:6384 为已有集群中任何节点



2.删除节点（删除顺序是先删从节点再删除主节点）
./redis-trib.rb del-node 127.0.0.1:6386 b51de0185386c80b73fbe87a68b1b580bb558b4e

可以通过登录Redis集群，执行 cluster nodes查看相关信息 



相关问题：
cannot load such file -- zlib 问题解决
原因
缺少zlib函式库
缺少ruby-zlib
解决
（未安装zlib时）下载安装zlib
我是使用源码安装，由是默认安装到/usr/local/lib，我选择使用root用户操作
安装版本zlib-1.2.11
下载软件包 wget http://www.zlib.net/zlib-1.2.11.tar.gz
解压缩软件包 tar -zxvf zlib-1.2.11.tar.gz
进入zlib源码目录 cd zlib-1.2.11/
配置 ./configure
make make
检查 make check
安装 make install
查看是否成功(目录中存在libz.a) find /usr/local/lib -name libz.a
安装 ruby-zlib
Ruby源码提供了该源码，直接找到对应目录安装
cd /root/ruby-2.6.5/ext/zlib
ruby ./extconf.rb
如果报错 checking for zlib.h... no 或 checking for deflateReset() in -lzlib... no
则 ruby ./extconf.rb --with-zlib-dir =/usr/local/zlib
make
如果报错 make: *** No rule to make target /include/ruby.h', needed byzlib.o'. Stop.
根据日志得知 zlib.o: $(top_srcdir)/include/ruby.h 去查看源码的确不存在变量值的话
在Makefile文档第一行，设置变量top_srcdir的路径
(我采用)用绝对/相对路径替换$(top_srcdir)
建议先备份Makefile
vim Makefile
: %s/$(top_srcdir)/..\/../g
：wq
如果上一步make报错，在修改后再次make
make install
