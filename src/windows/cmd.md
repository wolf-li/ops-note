

## 常用命令

### 系统管理

systeminfo 获取系统信息 
gpedit.msc 打开组策略编辑器
services.msc 管理本地服务
compmgmt.msc 打开计算机管理
taskmgr 打开任务管理器
msconfig 系统配置实用程序

### 网络相关

ipconfig 查看 IP 地址信息， /all 显示详细配置
ping 测试网络连接 ICMP
netstat 查看网络连接状态  netstat -an
tracert 跟踪数据包路径


### 文件操作

type \<filename\>  查看文件内容
dir 列出当前目录下的文件和文件夹
cd 切换目录
copy 复制文件
del 删除文件
mkdir 创建目录

### 系统维护

chkdsk 检查磁盘错误 chkdsk C:
sfc /scannow 扫描并修复系统
shutdown 关机或重启 shutdown /s 关机 shutdown /r 重启

### 其他实用命令

notepad 打开记事本
calc 打开计算器
cls 清屏
exit 退出 CMD
explorer . 在当前目录中打开 explorer 窗口n
color 02 第一个数字0是背景颜色，第二个数字2是字体颜色绿色
prompt {your want write}$G 自定义命令提示符文本
prompt 恢复默认命令提示符文本
title {stuff} 修改 cmd 窗口标题

### 骚操作

压缩文件图标为图片
copy /b image.extension+folder.zip image.extension

文件锁定
cipher /E

隐藏文件夹
attrib +h +s +r foldername
恢复隐藏文件夹
attrib -h -s -r foldername

显示所有连接过的 wifi 网络
netsh wlan show profile

显示某个 wifi 网络及密码
netsh wlan show profile "wifinetworkName" key=clear | findstr "Key Content"

查看所有 wifi 网络及密码
for /f "skip=9 token=1,2 delims=:" %i in ('netsh wlan show profiles') do @if "%j" NEQ "" (echo SSID:%j & netsh wlan show profiles %j key=clear | findstr "Key Content") & echo.

所有 wifi 网络及密码放入文件中
for /f "skip=9 token=1,2 delims=:" %i in ('netsh wlan show profiles') do @if "%j" NEQ "" (echo SSID:%j & netsh wlan show profiles %j key=clear | findstr "Key Content") >> wifipassword.txt

文件夹映射为驱动器
subst "驱动器符号如 s:" c:\filelocation
取消命令
subst /d "驱动器符号如 s:" 

curl 命令查看天气
curl wttr.in/beijing

短连接还原
curl --head --location "https://ntck.co/itprotv" | findstr Location

查看网站是否在线
curl -IsL https://networkchunk.com 

查看公网ip
curl checkip.amazonaws.com
