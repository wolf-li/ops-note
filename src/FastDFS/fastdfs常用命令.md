## 监控进程状态（状态为ACTIVE表示正常）

/usr/local/fastdfs/bin/fdfs_monitor /etc/fdfs/storage.conf

 
storage状态列表：

* FDFS_STORAGE_STATUS：INIT :初始化，尚未得到同步已有数据的源服务器

* FDFS_STORAGE_STATUS：WAIT_SYNC :等待同步，已得到同步已有数据的源服务器

* FDFS_STORAGE_STATUS：SYNCING :同步中

* FDFS_STORAGE_STATUS：DELETED :已删除，该服务器从本组中摘除

* FDFS_STORAGE_STATUS：OFFLINE :离线

* FDFS_STORAGE_STATUS：ONLINE :在线，尚不能提供服务

* FDFS_STORAGE_STATUS：ACTIVE :在线，可以提供服务

## 上传文件
fdfs_test [client.conf] upload [test file path]
结果
```
This is FastDFS client test program v5.11

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.
Please visit the FastDFS Home Page http://www.csource.org/ 
for more detail.

[2022-03-14 22:44:13] DEBUG - base_path=/data/fdfs/fastdfs/tracker, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
	server 1. group_name=, ip_addr=10.250.76.99, port=23000

group_name=group1, ip_addr=10.250.76.99, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/CvpMY2IvVL2ARbJMAAAAApiu_FM683.txt
source ip address: 10.250.76.99
file timestamp=2022-03-14 22:44:13
file size=2
file crc32=2561604691
example file url: http://10.250.76.99/group1/M00/00/00/CvpMY2IvVL2ARbJMAAAAApiu_FM683.txt
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/CvpMY2IvVL2ARbJMAAAAApiu_FM683_big.txt
source ip address: 10.250.76.99
file timestamp=2022-03-14 22:44:13
file size=2
file crc32=2561604691
example file url: http://10.250.76.99/group1/M00/00/00/CvpMY2IvVL2ARbJMAAAAApiu_FM683_big.txt
```

## 下载文件
/usr/bin/fdfs_download_file /etc/fdfs/client.conf group1/M00/00/00/CvpMY2IvVL2ARbJMAAAAApiu_FM683.txt

结果：
文件下载到当前目录下

## 删除文件
/usr/bin/fdfs_delete_file /etc/fdfs/client.conf group1/M00/00/00/CvpMY2IvVL2ARbJMAAAAApiu_FM683.txt



参考文件：
https://www.cnblogs.com/-yuanzheng/p/15185099.html
https://blog.csdn.net/m0_67402564/article/details/123304970

