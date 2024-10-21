1、安装环境   
yum install make cmake gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel -y   
2、解压文件   
3、创建 logs 目录防止编译失败   
mkdir -p /data/nginx/logs
4、执行：./configure --prefix=/data/nginx --with-http_stub_status_module --with-http_ssl_module --add-module=/data/nginx-1.15.3/nginx-upload-progress-module-master --with-stream   
5、执行：make && make install   
6、nginx的启动：/data/nginx/sbin/nginx   
7、nginx的停止：/data/nginx/sbin/nginx -s stop   
8、nginx配置文件热生效：/data/nginx/sbin/nginx -s reload   
9、添加环境变量   
echo "PATH=\$PATH:/data/nginx/sbin" > /etc/profile.d/nginx.sh     
source /etc/profile.d/nginx.sh     
   

