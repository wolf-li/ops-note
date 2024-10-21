# 一  上传搭建所需文件到服务器  
使用FTP工具把fastdfs-5.11.zip和libfastcommon.zip上传到服务器的/opt目录下(推荐在/home下放一份安装包备份)  
wget 下载  
# 二  开始搭建(简易单节点部署)  
## 1  使用Xmanager的远程终端链接服务器  
这里如果是对虚拟机进行操作，也可以直接使用虚拟机的终端。  
## 2  安装libfastcommon  
注意：在Centos7下和在Ubuntu下安装FastDFS是不同的，在Ubuntu上安装FastDFS需要安装libevent,而外Centos上安装FastDFS需要安装libfastcommon。  
我们以Centos为例进行FastDFS安装讲解。  
首先安装zip解压  
```
yum -y install unzip zip  
```
经过一系列过程后，出现Comlete!的字样，说明安装成功了。  
安装成功后解压libfastcommon.zip，这时进入/usr/local文件夹进行解压操作。  
  
```
cd /opt  
unzip libfastcommon.zip  
```
这是可以看到/usr/local文件夹下多了一个libfastcommon-master的文件夹  
接下来为了使用gcc命令，我们需要先安装gcc。否则在执行./make.sh时会报错的。  
  

`yum -y install gcc-c++  make`


安装成功后，进入libfastcommon-master的文件夹，先后执行./make.sh和./make.sh install  
  
```
cd /opt/libfastcommon-master  
./make.sh  && ./make.sh install  
```  
安装结束后，libfastcommon默认会被安装到/usr/lib64/libfastcommon.so但是FastDFS的主程序却在/usr/local/lib目录下，这个时候我们就要建立一个软链接了，实际上也相当于windows上的快捷方式。  
  
```
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so  
ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so  
ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so  
```  
## 3  安装FastDFS  
推荐新建如下三个目录  
/data/fdfs/fastdfs/tracker tracker服务目录  
/data/fdfs/fastdfs/storage storage服务目录  
/data/fdfs/fastdfs/storagedata文件存储目录  
以下是新建目录的命令：  
```
mkdir /data/fdfs/fastdfs  
mkdir /data/fdfs/fastdfs/tracker  
mkdir /data/fdfs/fastdfs/storage  
mkdir /data/fdfs/fastdfs/storagedata  
```  
进入/usr/local目录，解压安装包  
```
cd /opt  
unzip fastdfs-5.11.zip  
```
解压完后，发现目录下多了fastdfs-5.11这个文件夹  
进入这个文件夹，先后执行./make.sh和./make.sh install  
`./make.sh  && ./make.sh install  `
  
安装完成后，进入目录/etc/fdfs，执行下列语句，把client.conf.sample、storage.cof.sample、tracker.conf.sample复制一份。  
```
cp client.conf.sample client.conf  
cp storage.conf.sample storage.conf  
cp tracker.conf.sample tracker.conf  
```
至此FastDFS已经安装完毕，接下来的工作就是依次配置Tracker和Storage了。  
  
## 4  配置tracker  
在配置Tracker之前，首先需要创建Tracker服务器的文件路径，即用于存储Tracker的数据文件和日志文件等，我这里选择在/data/fdfs/fastdfs/tracker目录用于存放Tracker服务器的相关文件：  
  
接下来就要重新编辑上一步准备好的/etc/fdfs目录下的tracker.conf配置文件  
(为了修改配置文件，需要先安装vim)  

`yum -y install vim  `
`vim tracker.conf  `  
打开文件后依次做以下修改：  
```
disabled=false  #启用配置文件（默认启用）  
port=22122 #设置tracker的端口号，通常采用22122这个默认端口  
base_path=/data/fdfs/fastdfs/tracker  #设置tracker的数据文件和日志目录  
http.server_port=6666 #设置http端口号，默认为8080  
```
打开文件后，按“i”键，进行插入模式  
修改完成后，按“Ctrl+C”键，结束插入模式，按“shift+:”，然后输入“wq”，回车。保存退出。  
配置完成后就可以启动Tracker服务器了，但首先依然要为启动脚本创建软引用，因为fdfs_trackerd等命令在/usr/local/bin中并没有，  
  
而是在/usr/bin路径下：  
```
ln -s /usr/bin/fdfs_trackerd /usr/local/bin  
ln -s /usr/bin/stop.sh /usr/local/bin  
ln -s /usr/bin/restart.sh /usr/local/bin  

```
最后通过命令启动Tracker服务器：  
`service fdfs_trackerd start  `
  
如果启动命令执行成功，那么同时在刚才创建的tracker文件目录/data/fdfs/fastdfs/tracker中就可以看到启动后新生成的data和logs目录，tracker服务的端口也应当被正常监听。  
首先要安装net工具才可以使用netstat命令。  
  
```
yum install net-tools  
netstat -unltp|grep fdfs  
```  
netstat 参数  
-t或--tcp 显示TCP传输协议的连线状况。  
-u或--udp 显示UDP传输协议的连线状况。  
-n 直接使用IP地址，而不通过域名服务器。 快  
-l 显示监控中的服务器的Socket。  
-p 显示正在使用Socket的程序识别码和程序名称。  
类似命令  
  
`ss -tulnp  `
  
  
确认tracker正常启动后可以将tracker设置为开机启动，打开/etc/rc.d/rc.local并在其中加入以下配置：  
  
`service fdfs_trackerd start`  
centos的rc.local没有自启权限，通过以下命令给他增加自启权限。  
chmod +x /etc/rc.d/rc.local  
至此，Tracker配置成功。  
  
## 5  配置storage  
接下来修改/etc/fdfs目录下的storage.conf配置文件，打开文件后依次做以下修改：  
```
disabled=false #启用配置文件（默认启用）  
group_name=group1 #组名，根据实际情况修改  
port=23000 #设置storage的端口号，默认是23000，同一个组的storage端口号必须一致  
base_path=/data/fdfs/fastdfs/storage  #设置storage数据文件和日志目录  
store_path_count=1 #存储路径个数，需要和store_path个数匹配  
store_path0= /data/fdfs/fastdfs/storagedata  #实际文件存储路径  
tracker_server=192.168.36.135:22122 #tracker 服务器的 IP地址和端口号，如果是单机搭建，IP不要写127.0.0.1，否则启动不成功（此处的ip是我的CentOS虚拟机ip）  
http.server_port=8888 #设置 http 端口号
```
配置完成后同样要为Storage服务器的启动脚本设置软引用：  
`ln -s /usr/bin/fdfs_storaged /usr/local/bin ` 

接下来就可以启动Storage服务了：  
`service fdfs_storaged start ` 
如果启动成功，/data/fdfs/fastdfs/storage中就可以看到启动后新生成的data和logs目录，文件存储路径/data/fdfs/fastdfs/storagedata下会生成多级存储目录。端口23000也应被正常监听。  
data下有256个1级目录，每级目录下又有256个2级子目录，总共65536个文件，新写的文件会以hash的方式被路由到其中某个子目录下，然后将文件数据直接作为一个本地文件存储到该目录中。  
添加开机启动，打开/etc/rc.d/rc.local并将如下配置追加到文件中：  
`service fdfs_storaged start ` 
接下来测试storage服务器是否已经登记到 tracker服务器（也可以理解为tracker与storage是否整合成功），运行以下命令：  
`/usr/bin/fdfs_monitor /etc/fdfs/storage.conf  `
看到ip_addr = 192.168.36.135 (localhost.localdomain)  ACTIVE 字样即可说明storage服务器已经成功登记到了tracker服务器。  
  
## 6  模拟上传  
测试时需要设置客户端的配置文件，编辑/etc/fdfs目录下的client.conf 文件，打开文件后依次做以下修改：  
**base_path=/opt/fastdfs/tracker #tracker服务器文件路径  
tracker_server=192.168.36.135:22122 #tracker服务器IP地址和端口号  
http.tracker_server_port=6666 # tracker 服务器的 http 端口号，必须和tracker的设置对应起来  **

配置完成后就可以模拟文件上传了，先给/usr/local目录下放一张图片：ceshi.jpg  
\# 先可以测试普通文件传输 ，将下载下来的 fastdfs 作为测试目录  
\# 此软件有自带测试程序  
命令  
` /usr/bin/fdfs_test conf/client.conf upload /usr/include/stdlib.h  `
  
然后通过执行客户端上传命令尝试上传：  
/usr/bin/fdfs_upload_file  /etc/fdfs/client.conf  /usr/local/ceshi.jpg  
收到如下返回信息：  
group1/M00/00/00/wKgkh1qyO-qACziSAADmo2ljFnE803.jpg  
组名：group1
磁盘：M00
目录：00/00
文件名称：wKgkh1qyO-qACziSAADmo2ljFnE803.jpg  
进入/opt/fastdfs/storagedata/data/00/00，可以看到，文件上传成功。  
  
  
## 7  客户端访问tracker注意事项  
以下几个端口需要打开，否则无法正常访问tracker和stoarge  
22122、6666、23000、8888  
首先输入以下命令（以22122端口为例）  
firewall-cmd --permanent --zone=public --add-port=22122/tcp  


参数含义如下：  
--zone #作用域  
--add-port=22122/tcp  #添加端口，格式为：端口/通讯协议  
--permanent#永久生效，没有此参数重启后失效  
执行后需要重启防火墙生效  
systemctl restart firewalld.service  