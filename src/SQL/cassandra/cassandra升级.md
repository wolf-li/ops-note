# Cassandra 升级

>备注1: *本文升级方式适用于数据量较小，且使用程度不高时进行升级
>备注2:*  本文是 cassandra 3.11.3 升级为  3.11.13

## 准备活动

### 数据备份

#### 导出导入升级方式备份数据

##### 导出数据

```shell
#导出keyspace信息
cqlsh -u username -p password -e "DESC KEYSPACE 键空间" > /home/app/ospthings.cql

# keyspace 中所有表
cqlsh -u username -p password -e "DESC KEYSPACE 键空间" | grep "CREATE TABLE" | awk '{print $3}'

# 表数据导出
cqlsh -u username -p password -e "copy 键空间.表名 to '/data/cassandra_backup/xxx.csv' WITH HEADER = true;"
```

### 快照方式备份数据

#### 备份数据

```
nodetool snapshot -t 20220322 键空间
nodetool listsnapshots
```

#### 修改新版本相关配置与旧版本近似，具体参数灵活变通

比较修改下面的文件
>cassandra.yaml  
>cqlsh.py

数据库数据目录可以复制或者指向数据。

## 升级过程

1. 停止旧服务  
2. 复制源数据目录  
3. 启动新版本服务  
4. 测试  
   1） 查看新版本服务是否启动，集群状态 （ps  | nodetool status ）  
   2） 检查数据库状态，查看数据库日志有无报错 (system.log)  
   3） 创建数据库测试增删改查是否可用  

```
# 创建一个表空间
create keyspace mykeyspace with replication = {'class':'SimpleStrategy','replication_factor'}
# 创建一个表
create table cache ( 
    id int primary key, 
    type text,
    value text, 
    other text
);
# 插入一个数据
insert into cache (id, type, value, other) values (1, 'test', 'Mytest1', 'no');
# 修改一个数据
update cache set other = 'No' where id =1;
# 删除一个数据
delete from cache where id = 3;
```

## 升级失败立即回滚

测试过程过有任何为问题立即停止新服务，启动旧服务。
