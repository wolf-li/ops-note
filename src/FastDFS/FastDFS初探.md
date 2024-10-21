# 认识 fastDFS
FastDFS是一款开源的分布式文件系统，功能主要包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了文件大容量存储和高性能访问的问题。#FastDFS特别适合以文件为载体的在线服务，如图片、视频、文档等等。

### FastDFS 系统有三个角色：跟踪服务器(Tracker Server)、存储服务器(Storage Server)和客户端(Client)。

　　Tracker Server：跟踪服务器，主要做调度工作，起到均衡的作用；负责管理所有的 storage server和 group，每个 storage 在启动后会连接 Tracker，告知自己所属 group 等信息，并保持周期性心跳。

　　Storage Server：存储服务器，主要提供容量和备份服务；以 group 为单位，每个 group 内可以有多台 storage server，数据互为备份。

　　Client：客户端，上传下载数据的服务器，也就是我们自己的项目所部署在的服务器。

## FastDFS安装
### 一 、安装 FastDFS 前准备
本文使用源码安装，FastDFS是 C 编写的程序，需要编译。需要使用 gcc-c++，可选工具 git、unzip、wget。
安装相应软件 

```shell
yum -y install gcc-c++ git unzip wget
```

### 二 、安装  libfastcommon 和 FastDFS
需要的安装包：
*libfastcommon-1.0.36.tar.gz*
*fastdfs-5.11.tar.gz*

\## 提示：
\# 如果要升级或变换版本，请查阅相应的官方文档连接在本文最后。
\# FastDFS 5.11同以前版本相比将公共的一些函数等单独封装成了libfastcommon包， 所以在安装FastDFS之前我们还必须安装libfastcommon。

方法1 手工下载指定安装包
下载地址：
fastdfs-5.11.tar.gz
[gitee](https://gitee.com/fastdfs100/fastdfs/repository/archive/refs/tags/V5.11.tar.gz)
[github](https://github.com/happyfish100/fastdfs/archive/refs/tags/V5.11.tar.gz)

libfastcommon-1.0.36.tar.gz
[gitee](https://gitee.com/fastdfs100/libfastcommon/repository/archive/V1.0.35)
[github](https://github.com/happyfish100/libfastcommon/archive/refs/tags/V1.0.36.tar.gz)

使用FTP工具或scp命令 把fastdfs-5.11.zip和libfastcommon.zip上传到服务器的/opt目录下(推荐在/home下放一份安装包备份)
国内对 github 的封锁不建议使用github，可以使用 gitee 链接进行下载。
上传到服务器后。
1）先安装libfastcommon
首先解压压缩包
tar -xvfz libfastcommon*.tar.gz

解压后，进入解压后软件，安装软件
cd libfastcommon

\#使用下列命令进行软件安装
./make.sh clean && ./make.sh && ./make.sh install

安装结束后，libfastcommon默认会被安装到/usr/lib64/libfastcommon.so但是FastDFS的主程序却在/usr/local/lib目录下，这个时候我们就要建立一个软链接了，实际上也相当于windows上的快捷方式。

```shell
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
```

2）在进行 FastDFS 安装
创建数据目录方便以后运维
推荐新建如下三个目录
/data/fdfs/fastdfs/tracker       tracker服务目录
/data/fdfs/fastdfs/storage       storage服务目录
/data/fdfs/fastdfs/storagedata   文件存储目录
创建目录命令

```shell
mkdir -p /data/fdfs/fastdfs/{tracker,storage,storagedata}
```

解压缩下载的安装包

```shell
tar -xvfz fastdfs*.tar.gz
```

解压后，进入解压后软件，安装软件

```shell
cd fastdfs
```

\#使用下列命令进行软件安装

```shell
./make.sh clean && ./make.sh && ./make.sh install
```

安装完成后，进入目录/etc/fdfs，执行下列语句，把client.conf.sample、storage.cof.sample、tracker.conf.sample复制一份。

```shell
cp client.conf.sample client.conf
cp storage.conf.sample storage.conf
cp tracker.conf.sample tracker.conf
```

至此FastDFS已经安装完毕，接下来的工作就是依次配置Tracker和Storage了。


方法2 克隆仓库，回退版本
\# 下载 libfastcommon 源码并安装
[github address](https://github.com/happyfish100/libfastcommon.git)
[gitee address](https://gitee.com/fastdfs100/libfastcommon.git)

```shell
   git clone https://github.com/happyfish100/libfastcommon.git
   cd libfastcommon; git checkout V1.0.36
   ./make.sh clean && ./make.sh && ./make.sh install
```

\#下载 fastdfs 源码并安装
[github address](https://github.com/happyfish100/fastdfs.git)
[gitee address](   https://gitee.com/fastdfs100/fastdfs.git)

```shell
   git clone https://github.com/happyfish100/fastdfs.git
   cd fastdfs; git checkout V5.11
   ./make.sh clean && ./make.sh && ./make.sh install
   ```

安装完成后，进入目录/etc/fdfs，执行下列语句，把client.conf.sample、storage.cof.sample、tracker.conf.sample复制一份。
  
cp client.conf.sample client.conf
cp storage.conf.sample storage.conf
cp tracker.conf.sample tracker.conf
  
至此FastDFS已经安装完毕，接下来的工作就是依次配置Tracker和Storage了。

### 三、 配置
#### a. tracker 配置
在配置Tracker之前，首先需要创建Tracker服务器的文件路径，即用于存储Tracker的数据文件和日志文件等，我这里选择在/data/fdfs/fastdfs/tracker目录用于存放Tracker服务器的相关文件：

修改配置文件 (/etc/fdfs/tracker.conf)
方法1 使用 vi，vim，nano 等文件编辑器
这里进行 vi 介绍，其他读者可自行百度，或查看 man 手册、help等。

打开文件
vi tracker.conf
对配置进行修改

```
disabled=false  #启用配置文件（默认启用）
port=22122 #设置tracker的端口号，通常采用22122这个默认端口
base_path=/data/fdfs/fastdfs/tracker  #设置tracker的数据文件和日志目录
http.server_port=6666 #设置http端口号，默认为8080
```

打开文件后，按“i”键，进行插入模式
修改完成后，按“Ctrl+C”键，结束插入模式，按“shift+:”，然后输入“wq”，回车。保存退出。

方法2 使用 sed 进行替换 （tracker.conf 要根据文件具体路径进行修改）

```shell
 sed -i "s:base_path=/home/yuqing/fastdfs:base_path=/data/fdfs/fastdfs/tracker:" /etc/fdfs/tracker.conf
 sed -i "s/http.server_port=8080/http.server_port=6666/" /etc/fdfs/tracker.conf
```


配置完成后就可以启动Tracker服务器了，但首先依然要为启动脚本创建软引用，因为fdfs_trackerd等命令在/usr/local/bin中并没有，
而是在/usr/bin路径下：

```shell
ln -s /usr/bin/fdfs_trackerd /usr/local/bin
ln -s /usr/bin/stop.sh /usr/local/bin
ln -s /usr/bin/restart.sh /usr/local/bin
```

最后通过命令启动Tracker服务器：

```shell
service fdfs_trackerd start
```

还可以使用软件自带命令启动

```shell
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
```

如果启动命令执行成功，那么同时在刚才创建的tracker文件目录/data/fdfs/fastdfs/tracker中就可以看到启动后新生成的data和logs目录，tracker服务的端口也应当被正常监听。
使用系统自带的 ss 工具进行查看

```shell
ss -tulnp |grep fdfs
```

会出现类似下面的显示，（端口设置的不同，显示结果有所不同）

```
tcp    LISTEN     0      128       *:22122  
```

确认tracker正常启动后可以将tracker设置为开机启动，打开/etc/rc.d/rc.local并在其中加入以下配置：

```shell
service fdfs_trackerd start
```

centos的rc.local没有自启权限，通过以下命令给他增加自启权限。

```shell
chmod +x /etc/rc.d/rc.local
```

至此，Tracker配置成功。

#### b. storage 配置
修改配置文件 （/etc/fdfs/storage.conf）

方法1 使用 vi，vim，nano 等文件编辑器
打开文件后依次做以下修改：

```
disabled=false            #启用配置文件（默认启用）
group_name=group1         #组名，根据实际情况修改
port=23000                #设置storage的端口号，默认是23000，同一个组的storage端口号必须一致
base_path=/data/fdfs/fastdfs/storage   #设置storage数据文件和日志目录
store_path_count=1        #存储路径个数，需要和store_path个数匹配
store_path0= /data/fdfs/fastdfs/storagedata     #实际文件存储路径
tracker_server=192.168.36.135:22122       #tracker 服务器的 IP地址和端口号，如果是单机搭建，IP不要写127.0.0.1，否则启动不成功（此处的ip是我的CentOS虚拟机ip）
http.server_port=8888     #设置 http 端口号
```
打开文件后，按“i”键，进行插入模式
修改完成后，按“Ctrl+C”键，结束插入模式，按“shift+:”，然后输入“wq”，回车。保存退出。

方法2 使用 sed

```shell
 sed -i "s:base_path=/home/yuqing/fastdfs:base_path=/data/fdfs/fastdfs/storage:" storage.conf
 sed -i "s:store_path0=/home/yuqing/fastdfs:store_path0= /data/fdfs/fastdfs/storagedata:" storage.conf
 ip=$(ip add | grep inet | grep -v 127 | awk '{print $2}' | sed s:/16::)
 sed -i "s/tracker_server=192.168.209.121:22122/tracker_server=$ip:22122/" storage.conf
```

配置完成后同样要为Storage服务器的启动脚本设置软引用：

```shell
ln -s /usr/bin/fdfs_storaged /usr/local/bin
```

接下来就可以启动Storage服务了：

```shell
service fdfs_storaged start
```

如果启动成功，/data/fdfs/fastdfs/storage中就可以看到启动后新生成的data和logs目录，文件存储路径/data/fdfs/fastdfs/storagedata下会生成多级存储目录。端口23000也应被正常监听。
data下有256个1级目录，每级目录下又有256个2级子目录，总共65536个文件，新写的文件会以hash的方式被路由到其中某个子目录下，然后将文件数据直接作为一个本地文件存储到该目录中。
添加开机启动，打开/etc/rc.d/rc.local并将如下配置追加到文件中：

```shell
service fdfs_storaged start
```

接下来测试storage服务器是否已经登记到 tracker服务器（也可以理解为tracker与storage是否整合成功），运行以下命令：

```shell
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```

看到ip_addr = 192.168.36.135 (localhost.localdomain)  ACTIVE 字样即可说明storage服务器已经成功登记到了tracker服务器。

## 四 测试验证
测试时需要设置客户端的配置文件，编辑/etc/fdfs目录下的client.conf 文件，打开文件后依次做以下修改：
base_path=/opt/fastdfs/tracker #tracker服务器文件路径
tracker_server=192.168.36.135:22122 #tracker服务器IP地址和端口号
http.tracker_server_port=6666 # tracker 服务器的 http 端口号，必须和tracker的设置对应起来

我将解压缩的 FastDFS 目录下的 stop.sh 文件作为示例
通过执行客户端上传命令尝试上传：

```shell
/usr/bin/fdfs_upload_file  /etc/fdfs/client.conf /opt/fastdfs/stop.sh
```

\# 有以下显示

```
group1/M00/00/00/rBEAAmD2l0CAM5s0AAAGkCLo-iI1604.sh
```

可以看到，文件上传成功。

## 五 注意事项
以下几个端口需要打开，否则无法正常访问tracker和stoarge
22122、6666、23000、8888
首先输入以下命令（以22122端口为例）
firewall-cmd --permanent --zone=public --add-port=22122/tcp
 
 
参数含义如下：
--zone #作用域  
--add-port=22122/tcp  #添加端口，格式为：端口/通讯协议  
--permanent   #永久生效，没有此参数重启后失效  
执行后需要重启防火墙生效  
systemctl restart firewalld.service  



## 参考文献：
[github官方安装手册](https://github.com/happyfish100/fastdfs/blob/master/INSTALL)  
[gitee 官方安装手册](https://gitee.com/fastdfs100/fastdfs/blob/V5.11/INSTALL)  
[用FastDFS一步步搭建文件管理系统 ](https://www.cnblogs.com/chiangchou/p/fastdfs.html)  
[Docker中搭建FastDFS](https://www.cnblogs.com/niceyoo/p/13511082.html)