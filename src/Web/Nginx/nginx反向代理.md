## nginx反向代理隐藏响应头
经常会用nginx做反向代理到后端应用服务器，如代理到java、php等应用。有个问题是返回给客户端响应头中可能包含一些后端信息，虽然不影响用户体验，但还是泄露一点信息。

比如我nginx反向代理到后端的java应用的8888端口，在响应头中被显示出来了
可以用 proxy_hide_header  隐藏响应头中的某些信息

`proxy_hide_header X-Application-Context;    # 配置在你的proxy`

## 反向代理配置模板

location / {
    proxy_set_header Host $host;
    proxy_set X-Real-IP $remote_addr;
    proxy_pass http://;
}

location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version  1.1;
    proxy_cache_bypass  $http_upgrade;

    proxy_set_header Upgrade           $http_upgrade;
    proxy_set_header Connection        "upgrade";
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host  $host;
    proxy_set_header X-Forwarded-Port  $server_port;
}

proxy_http_version 1.1定义用于代理的HTTP协议版本，默认情况下将其设置为1.0。对于Websocket和keepalive连接，您需要使用1.1版。
proxy_cache_bypass $http_upgrade设置websocket不从缓存中获取响应，而是直接通过应用。
Upgrade $http_upgrade和Connection "upgrade"如果您的应用程序使用Websockets，则这些字段是必填字段。
X-Real-IP $remote_addr将真实的客户端地址转发到应用，如果没有设置，你应用获取到将会是Nginx服务器IP地址。
X-Forwarded-For $proxy_add_x_forwarded_for转发客户端请求头的X-Forwarded-For字段到应用。
如果客户端请求头中不存在X-Forwarded-For字段，则$proxy_add_x_forwarded_for变量等同于$remote_addr变量
X-Forwarded-Proto $scheme这将会转发客户端所使用的HTTP协议或者是HTTPS协议。
​​X-Forwarded-Host $host转发客户端请求的原始主机到应用。X-Forwarded-Port $server_port定义客户端请求的原始端口。


kill -QUIT 132505