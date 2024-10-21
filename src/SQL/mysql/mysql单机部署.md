# mysql bin文件单机部署

```shell
useradd mysql 
mkdir -p /var/lib/mysql
tar xzf mysql-5.7.35-el7-x86_64.tar.gz -C /data
mv /data/mysql-5.7.35-el7-x86_64/ /data/mysql-5.7.35
mkdir -p /data/mysql-5.7.35/data
chown -R mysql:mysql /var/lib/mysql /data/mysql-5.7.35
yum install -y autoconf


cat << EOF > /etc/my.cnf
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
socket=/var/lib/mysql/mysql.sock
[mysqld]
skip-name-resolve
#设置3306端口
port = 3306
socket=/var/lib/mysql/mysql.sock
# 设置mysql的安装目录
basedir=/data/mysql-5.7.35
# 设置mysql数据库的数据的存放目录
datadir=/data/mysql-5.7.35/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 表名存储为给定的大小和比较是区分大小写
lower_case_table_names=0
max_allowed_packet=16M
explicit_defaults_for_timestamp=true
EOF

chown 644 /etc/my.cnf

sed -i "s,/usr/local/mysql,/data/mysql-5.7.35," /data/mysql-5.7.35/support-files/mysql.server

chown -R mysql:mysql /data/mysql-5.7.35

cd /data/mysql-5.7.35/bin
./mysqld --initialize --user=mysql --basedir=/data/mysql-5.7.35 --datadir=/data/mysql-5.7.35/data/

# 获取密码  nagwCaniy1,j

cd /data/mysql-5.7.35/support-files
cp ./mysql.server /etc/rc.d/init.d/mysqld


chmod +x /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
chkconfig --list mysqld
service mysqld start

echo "export PATH=/data/mysql-5.7.35/bin:\$PATH" >> /etc/profile
source /etc/profile

# 登录修改 root 密码
mysql -uroot -p
alter user user() identified by "Pansoft@2022"; 
flush privileges;
# 设置远程访问
use mysql; 
update user set user.Host='%' where user.User='root'; 
flush privileges;

firewall-cmd --add-port=3306/tcp --permanent
firewall-cmd --reload
```

# mysql 容器部署
```
# 拉取镜像
docker pull mysql:5.6.35
# 创建目录存储容器内配置文件
mkdir -p /data/mysql/{conf,data}
# 启动容器
docker run -p 13306:3306 --name my-mysql -v /data/mysql/conf:/etc/mysql -v /data/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6.35
# 启动容器（没有配置文件）
docker run -p 13306:3306 --name osp-mysql -v /data/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d  mysql:5.7.44


# 查看容器启动情况
docker ps|grep mysql
# 登录容器
docker exec -it my-mysql bash
# 登录 mysql 
root@5291ed3fe987:/# mysql -uroot -p --default-character-set=utf8
# 查看数据库
mysql> show databases;
# 设置 root 用户远程登录数据库
mysql> use mysql
# 设置root用户在任何地方进行远程登录，并具有所有库任何操作权限，（公司绝对不能这么做，暴露的攻击面太大），这里只是做测试。
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
# 刷新权限
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
# 退出mysql 
mysql> exit
Bye

# 使用 docker 命令在容器外部执行 mysql 相关命令
docker exec -it <container-name|id> sh -c 'mysql -u<username> -p<password> -e "sql command"'
# 执行 sql 文件
# 拷贝文件到容器中
docker cp <file-path> <container-name>:/tmp/  
# 判断文件是否存在
docker exec -it <container-name|id> /bin/sh -c 'if [[ ! -f <file-path> ]];then echo "not exist";else echo "exist";fi'
# 在外部执行加载sql 文件
docker exec -it <container-name|id> /bin/sh -c 'mysql -u<username> -p<password> -e "source <sql-file-path>"'

```
```shell
容器ip获取: docker inspect --format='{{.NetworkSettings.IPAddress}}' \<container-ID\>
```

