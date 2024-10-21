# postgresql 升级
注意：升级前先备份，联系业务
## 小版本升级
PostgreSQL每次的小版本升级不会改变内部的存储格式，不会改变数据目录， 并且总是向上兼容同一主版本， 例如9.6.2与9.6.1总是兼容的， 以此类推，9.6.3与9.6.2也是兼容的，无论他们之间跨越了几个小版本。 升级小版本也很简单， 只需安装新的可执行文件， 并重新启动数据库实例。

本次文档演示的版本为11.1升级到11.7


参考文章：
1. https://blog.csdn.net/m15217321304/article/details/108405660

## 大版本升级
### 原理：pg_upgrade

参考文章：
1. https://www.postgresql.org/docs/11/pgupgrade.html
2. https://www.cnblogs.com/hmwh/p/11372649.html


## yum 升级软件包
```
yum update -y *.rpm

# 修改配置文件夹
 vi /usr/lib/systemd/system/postgresql-11.service
  
# 将其中的Environment=PGDATA=/var/lib/pgsql/11/data/修改为
Environment=PGDATA=/data/pgsql/11/data/
 
# 执行下面的命令
/usr/pgsql-11/bin/postgresql-11-setup initdb
注意 ,如果不修改会默认安装到/var/lib/pgsql/11/data

systemctl daemon-reload
systemctl restart postgresql-11
```
### 集群升级
1. 数据备份
2. 集群停机
systemctl stop postgresql-11

3. 升级postgresql

