 # nginx反向代理网站镜像
 某些公司会墙特定网站，如果你有一个可访问的域名和服务器，就可以通过nginx反向代理来来解决这些问题。比如现在我们用mirror.example.com镜像www.baidu.com，以下是详细操作。

一、DNS里添加A记录，新增子域名，如：mirror.example.com

二、nginx里新增解析文件

（一）http去镜像http
```shell
server {
        listen 9999;
        server_name mirror.example.com;
        charset utf-8;
        location / {
          proxy_pass http://www.baidu.com;
          proxy_set_header Accept-Encoding deflate;
          sub_filter_once off;
          sub_filter www.baidu.com mirror.example.com;
   
          proxy_redirect off;
          proxy_set_header        X-Real-IP       $remote_addr;
          proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
          proxy_max_temp_file_size 0;
          proxy_connect_timeout 90;
          proxy_send_timeout 90;
          proxy_read_timeout 90;
          proxy_buffer_size 4k;
          proxy_buffers 4 32k;
          proxy_busy_buffers_size 64k;
          proxy_temp_file_write_size 64k;
        }
  }
```
（二）https去镜像https
```shell
server {
    listen 9999 ssl;
    server_name  mirror.example.com;
    ssl_certificate /usr/local/ssl/nginx.crt;       #证书公钥
    ssl_certificate_key  /usr/local/ssl/nginx.key;  #证书私钥
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDH:AESGCM:HIGH:!RC4:!DH:!MD5:!3DES:!aNULL:!eNULL;
    ssl_prefer_server_ciphers on;
    proxy_ssl_server_name on;
    # 下面这段location配置是关键
    location / {
       sub_filter www.baidu.com mirror.example.com;
       sub_filter_once off;
       proxy_ssl_session_reuse off;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header Referer https://www.baidu.com;
       proxy_set_header Host www.baidu.com;
       proxy_pass https://www.baidu.com;
       proxy_set_header Accept-Encoding "";
    }
}
```
三、重启nginx