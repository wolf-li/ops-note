一、数据的导入导出
mongoexport -h 127.0.0.1 --port 20001 -uxxx -pxxx -d xxx -c mobileIndex -o XXX.txt 
mongoimport -h 127.0.0.1 --port 20001 -uxxx -pxxx -d xxx -c mobileIndex --file XXX.txt
二、数据迁移
1、迁移复制集当中的成员
关闭 mongod 实例,为了确保安全关闭,使用 shutdown 命令；
将数据目录(即 dbPath )转移到新机器上；
在新机器上启动 mongod，其中节点的数据目录为copy的文件目录 ；
连接到复制集当前的主节点上；
如果新节点的地址发生变化,使用 rs.reconfig() 更新 复制集配置文档 ； 举例,下面的命令过程将成员中位于第 2 位的地址进行更新:

cfg = rs.conf()
cfg.members[2].host = "127.0.0.1:27017"
rs.reconfig(cfg)

使用 rs.conf()
确认使用了新的配置. 等待所有成员恢复正常,使用 rs.status()
检测成员状态。

2、迁移复制集主节点
在迁移主节点的时候,需要复制集选举出一个新的主节点,在进行选举的时候,复制集将读写,通常,这只会持续很短的时间,不过,应该尽可能在影响较小的时间段内迁移主节点.

主节点降级,以使得正常的 failover开始.要将主节点降级,连接到一个主节点,使用 replSetStepDown
方法或者使用rs.stepDown()
方法,下面的例子使用了 rs.stepDown()
方法进行降级:
rs.stepDown()

等主节点降级为从节点,另一个成员成为 PRIMARY
之后,可以按照 “迁移复制集的一个成员”迁移这个降级了的节点.可以使用 rs.status()
来确认状态的改变。
3、从复制集其他节点恢复数据
MongoDB 通过复制集能保证高可靠的数据存储，通常生产环境建议使用「3节点复制集」，这样即使其中一个节点崩溃了无法启动，我们可以直接将其数据清掉，重新启动后，以全新的 Secondary 节点加入复制集，或者是将其他节点的数据复制过来，重新启动节点，它会自动的同步数据，这样也就达到了恢复数据的目的。

关闭需要数据同步的节点
docker stop node;  # docker环境中
db.shutdownServer({timeoutSecs: 60}); # 非docker环境

拷贝目标节点机器的数据存储目录(/dbPath)到当前机器的指定目录。
scp 目标节点 shard/data -> 当前节点 shard/data

当前节点以复制过来的数据文件启动节点
将新的节点添加到复制集
# 进入复制集的主节点，执行添加新的节点命令
rs.add("hostNameNew:portNew"); 
# 等待所有成员恢复正常,检测成员状态
rs.status();
# 移除原来的节点
rs.remove("hostNameOld>:portOld"); 

三、MongoDB线上问题场景解决
1、MongoDB 新建索引导致库被锁
问题说明：某线上千万级别集合，为优化业务，直接执行新建索引命令，导致整个库被锁，应用服务出现不可用。

解决方案：找出此操作进程，并且杀死。改为后台新建索引，速度很会慢，但是不会影响业务，该索引只会在新建完成之后，才会生效；

# 查询运行时间超过200ms操作     
db.currentOp({"active" : true,"secs_running" : { "$gt" : 2000 }}) ；
# 杀死执行时间过长操作操作
db.killOp(opid)
# 后台新建索引
db.collectionNmae.ensureIndex({filedName:1}, {background:true});

2、MongoDB没有限制内存，导致实例退出
问题说明：生产环境某台机器启动多个mongod实例，运行一段时间过后，进程莫名被杀死；

解决方案：现在MongoDB使用WiredTiger作为默认存储引擎，MongoDB同时使用WiredTiger内部缓存和文件系统缓存。从3.4开始，WiredTiger内部缓存默认使用较大的一个：50％（RAM - 1 GB），或256 MB。 例如，在总共4GB RAM的系统上，WiredTiger缓存将使用1.5GB的RAM（）。相反，具有总共1.25 GB RAM的系统将为WiredTiger缓存分配256 MB，因为这超过总RAM的一半减去1千兆字节（）。0.5 * (4 GB - 1GB) = 1.5 GB``0.5 * (1.25 GB - 1 GB) = 128 MB < 256 MB。如果一台机器存在多个实例，在内存不足的情景在，操作系统会杀死部分进程；

# 要调整WiredTiger内部缓存的大小，调节cache规模不需要重启服务，我们可以动态调整：
db.adminCommand( { "setParameter": 1, "wiredTigerEngineRuntimeConfig": "cache_size=xxG"})

3、MongoDB删除数据，不释放磁盘空间
问题说明：在删除大量数据(本人操作的数据量在2000万+)的情景下，并且在生产环境中请求量较大，此时机器的cpu负载会显得很高，甚至机器卡顿无法操作，这样的操作应该谨慎分批量操作；在删除命令执行结束之后，发现磁盘的数据量大小并没有改变。

解决方案：

方案一：我们可以使用MongoDB提供的在线数据收缩的功能，通过Compact命令db.collectionName.runCommand("compact")
进行Collection级别的数据收缩，去除集合所在文件碎片。此命令是以Online的方式提供收缩，收缩的同时会影响到线上的服务。为了解决这个问题，可以先在从节点执行磁盘整理命令，操作结束后，再切换主节点，将原来的主节点变为从节点，重新执行Compact命令即可。

方案二：使用从节点重新同步，secondary节点重同步，删除secondary节点中指定数据，使之与primary重新开始数据同步。当副本集成员数据太过陈旧，也可以使用重新同步。数据的重新同步与直接复制数据文件不同，MongoDB会只同步数据，因此重同步完成后的数据文件是没有空集合的，以此实现了磁盘空间的回收。

针对一些特殊情况，不能下线secondary节点的，可以新增一个节点到副本集中，然后secondary就自动开始数据的同步了。总的来说，重同步的方法是比较好的，第一基本不会阻塞副本集的读写，第二消耗的时间相对前两种比较短。

若是primary节点，先强制将之变为secondary节点，否则跳过此步骤：rs.stepdown(120)；
然后在primary上删除secondary节点：rs.remove("IP:port");
删除secondary节点dbpath下的所有文件
将节点重新加入集群，然后使之自动进行数据的同步：rs.add("IP:port");
等数据同步完成后，循环1-4的步骤可以将集群中所有节点的磁盘空间释放
4、MongoDB机器负载极高
问题说明：此情景是在客户请求较大的情景性，由于部署MongoDB的机器包含一主一从，MongoDB使得IO100%，数据库阻塞，出现大量慢查询，进而导致机器负载极高，应用服务完全不可用。

解决方案：在没有机器及时扩容的状况下，首要任务便是减小机器的IO，在一台机器出现一主一从，在大量数据写入的情况下，会互相抢占IO资源。于是此时摒弃了MongoDB高可用的特点，摘掉了复制集当中的从节点，保证每台机器只有一个节点可以占用磁盘资源。之后，机器负载立马下来，服务变为正常可用状态，但是此时MongoDB无法保证数据的完整性，一旦有主节点挂掉便会丢失数据。此方案只是临时方法，根本解决是可以增加机器的内存、使用固态硬盘，或者采用增加分片集来减少单个机器的读写压力。

# 进入主节点，执行移除成员的命令
rs.remove("127.0.0.1:20001");
# 注意：切勿直接关停实例

5、MongoDB分片键选择不当导致热读热写
问题说明：生产环境中，某一集合的片键使用了与_id生成方式相似，含有时间序列的字段作为升序片键，导致数据写入时都在一个数据块，随着数据量增大，会造成数据迁移到前面的分区，造成系统资源的占用，偶尔出现慢查询。

解决方案：临时方案设置数据迁移的窗口，放在在正常的时间区段，对业务造成影响。根本解决是更换片键。

# 连接mongos实例，执行以下命令
db.settings.update({ _id : "balancer" }, { $set : { activeWindow : { start : "23:00", stop : "4:00" } } }, true )；
# 查看均衡窗口
sh.getBalancerWindow()；


## 查看数据库报错 [js] Error: listDatabases failed:
```
rs.secondaryOk()
```