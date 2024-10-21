## 卸载
### 停止服务
```
systemctl list-units | grep postgresql | awk '{print $1}' | xargs systemctl stop {}
```
### 卸载rpm 包
```
#rpm 包安装有依赖卸载时最好按照顺序卸载
rpm -e postgresql11-server
rpm -e postgresql11
rpm -e libicu
rpm -e postgresql11-libs
```
删除用户
userdel -r postgres
删除文件
find / -name pgsql | xargs -i rm -rf {}



