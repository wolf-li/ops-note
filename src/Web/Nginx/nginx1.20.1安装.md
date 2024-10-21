# nginx 1.20.1 安装  
## 软件安装包下载  
http://nginx.org/download/nginx-1.20.1.tar.gz  
  
## 准备相应依赖软件工具  
```shell  
yum -y install pcre pcre-devel openssl openssl-devel zlib zlib-devel  gcc-c++ make cmake gcc

yum install gcc gcc-c++ pcre-devel zlib-devel make unzip gd-devel perl-ExtUtils-Embed libxslt-devel openssl-devel perl-Test-Simple
```  
  
## 1.解压缩软件包  
```  
tar xzf /home/app/nginx-1.20.1.tar.gz -C /data  
```  
  
## 2. 安装选项配置编译  
```  
./configure --prefix=/data/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre  --with-http_v2_module
make && make install  
```  
  
## 3. 添加环境变量  
```  
echo "PATH=\$PATH:/data/nginx/sbin" >> /etc/profile.d/nginx.sh  
source /etc/profile.d/nginx.sh  
```  
  
## 4. 配置 server 文件，使用 systemctl 对 nginx 进行管理  
文件路径  
/etc/systemd/system/nginx.service  
  
文件内容  
```  
[Unit]  
Description=The NGINX HTTP and reverse proxy server  
After=syslog.target network-online.target remote-fs.target nss-lookup.target  
Wants=network-online.target  
  
[Service]  
Type=forking  
PIDFile=/data/nginx/logs/nginx.pid 
ExecStartPre=/data/nginx//sbin/nginx -t  
ExecStart=/data/nginx//sbin/nginx  
ExecReload=/data/nginx//sbin/nginx -s reload  
ExecStop=/bin/kill -s QUIT $MAINPID  
PrivateTmp=true  
  
[Install]  
WantedBy=multi-user.target  
```  
重载systemd配置文件

systemctl daemon-reload

启动服务

systemctl start nginx.service

开机启动

systemctl enable nginx.service

## 5. 测试  
测试配置文件是否有问题  
`nginx -t`  
启动 nginx  
`nginx`  
测试nginx 是否可以提供 web 服务  
```  
curl -I http://127.0.0.1  
HTTP/1.1 200 OK  
Server: nginx/1.20.1  
Date: Mon, 09 Aug 2021 18:50:36 GMT  
Content-Type: text/html  
Connection: keep-alive  
```  

## http2 使用
编译 nginx 软件包 要添加 --with-http_v2_module 模块，且需要 启用 https 后才可以使用 http2

检测是否使用 http2
浏览器 F12 右键添加协议