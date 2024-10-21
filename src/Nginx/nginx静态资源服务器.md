# 静态资源服务器配置

静态资源是指非服务器运行动态生成的文件，主要包括浏览器端渲染（html、css、js）、图片（jpeg、gif、png）、视频文件（flv、mpeg）、其他文件（TXT等任意下载文件）。

## nginx 配置文件

> server {
>         listen 80;
>         server_name 127.0.0.1;
> 
>         auth_basic "who are you?";
>         auth_basic_user_file /data/nginx/conf/htpasswd.users;
> 
> 
>         location  /ops {
>                alias   /data/ops/;
>                autoindex on; #开启目录浏览
>                autoindex_format html; #以html风格将目录展示在浏览器中
>                autoindex_exact_size off; #切换为 off 后，以可读的方式显示文件大小，单位为 KB、MB 或者 GB
>                autoindex_localtime on;  # 同步本地时间，默认关闭
>                charset utf-8,gbk; #展示中文文件名
>         }
> 
>         error_page 403 404 500 502 503 504 /error.html;
>         #proxy_intercept_errors on;
> 
>         location /error.html {
>             root   /data/nginx/html;
>             internal;
>         }
>     }

配置用名密码
安装工具
yum  -y install httpd-tools

htpasswd -c 文件路径 用户名
