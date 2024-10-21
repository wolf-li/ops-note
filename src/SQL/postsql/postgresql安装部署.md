### 一、环境准备

版本：

端口号为5432

##### 1.创建用户 组以及对应目录

```sh
# 新增用户和组 -r为内部用户
useradd postgres 
# 创建目录
mkdir -p /data/pgsql/11/data

chown -R postgres /data/pgsql
```

 

##### 2.安装依赖包

```sh
yum -y install gcc gcc-c++ bzip2 gzip zip unzip openssl openssl-devel zlib-devel zlib -y

yum install readline-devel -y

yum install libicu -y  #这个必须有，解决报错 libicui18n.so.50: cannot open shared object file: No such file or directory
```

##### 3.配置环境变量

```sh
# 编辑环境变量
vi /etc/profile

# 添加
export PATH=$PATH:/usr/pgsql-11/bin

# 使修改生效
source /etc/profile
```

### 二、服务安装

> 这里采用的离线安装方式

链接：https://pan.baidu.com/s/1qNCzUolozO5oIFJYAo_BzA 
提取码：0218

上传值/tmp目录下

```sh

rpm -ivh *.rpm --force --nodeps

# 修改配置文件夹
 vi /usr/lib/systemd/system/postgresql-11.service
  
# 将其中的Environment=PGDATA=/var/lib/pgsql/11/data/修改为
Environment=PGDATA=/data/pgsql/11/data/
 
# 执行下面的命令
/usr/pgsql-11/bin/postgresql-11-setup initdb
注意 ,如果不修改会默认安装到/var/lib/pgsql/11/data


systemctl start postgresql-11
systemctl enable postgresql-11.service

```



### 三、配置使用

```sh
su postgres    //yum安装的默认创建一个'postgres'用户
psql -U postgres    //  进入postgres数据库
修改'postgres'用户的登录密码
alter user postgres with password 'lA#=C9zZt-Y)';
```



修改/data/pgsql/11/data/pg_hba.conf配置文件.

`vi /data/pgsql/11/data/pg_hba.conf`

> 如果需要配置在非postgres用户下访问登录,需要将第pg_hba.conf中第一行local中的验证方式更改,可按需更改为md5(需要密码验证)或trust(无需密码验证).

```sh
host    all             all             0.0.0.0/0               md5
```
all 都需要配置，psql 才可以远程登录

修改postgresql.conf



修改防火墙

```sh
firewall-cmd --permanent --add-port=5432/tcp

systemctl restart firewalld

# 查看防火墙所有开放端口
firewall-cmd --zone=public --list-ports
```

