nginx 配置文件写法
```
server {
    listen 80;
    server_name www.域名.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent; 
}
server {
    listen 443;
    server_name www.域名.com;
    root /home/wwwroot;
    ssl on;
    ssl_certificate /etc/nginx/certs/server.crt;
    ssl_certificate_key /etc/nginx/certs/server.key;
    ....
}
```