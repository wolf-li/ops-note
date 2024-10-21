# nginx 优化  

## 安全优化  

### 1. 隐藏版本号  

```shell  
curl -I url     # 查看web返回头  
HTTP/1.1 200 OK  
Server: nginx/1.15.3  
Date: Tue, 03 Aug 2021 06:57:56 GMT  
Content-Type: text/html  
Content-Length: 6  
Last-Modified: Tue, 03 Aug 2021 05:25:33 GMT  
Connection: keep-alive  
ETag: "6108d34d-6"  
Accept-Ranges: bytes  
```  

修改配置文件（nginx.conf）隐藏版本号  

```shell  
http{  
 server_tokens off;                            #在http下面手动添加这么一行 (默认是 on)  
 … …  
}  
```  

平滑启动 nginx  

sbin/nginx -s reload  
  
检查 curl -I url

```shell  
HTTP/1.1 200 OK  
Server: nginx  
Date: Tue, 03 Aug 2021 07:54:37 GMT  
Content-Type: text/html  
Content-Length: 612  
Last-Modified: Tue, 03 Aug 2021 05:02:30 GMT  
Connection: keep-alive  
ETag: "6108cde6-264"  
```  

### 2. 拒绝非法请求方法  

该协议中定义了很多方法，可以让用户连接服务器，获得需要的资源。但实际应用中一般仅需要get和post。  
|序号| 方法| 描述|  
|--|--|--|  
1| GET| 请求指定的页面信息，并返回实体主体。  
2| HEAD| 类似于 GET 请求，只不过返回的响应中没有具体的内容，用于获取报头  
3| POST| 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立和/或已有资源的修改。  
4| PUT| 从客户端向服务器传送的数据取代指定的文档的内容。  
5| DELETE| 请求服务器删除指定的页面。  
6| CONNECT| HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。  
7| OPTIONS| 允许客户端查看服务器的性能。  
8| TRACE| 回显服务器收到的请求，主要用于测试或诊断。  
9| PATCH| 是对 PUT 方法的补充，用来对已知资源进行局部更新 。  
  
未修改服务器配置前，客户端使用不同请求方法测试：  
curl -i -X GET  <http://192.168.4.7>            #正常  
curl -i -X HEAD <http://192.168.4.7>            #正常  
  
> curl命令选项说明：  
> -i选项：访问服务器页面时，显示HTTP的头部信息  
> -X选项：指定请求服务器的方法  
通过如下设置配置文件（nginx.conf）可以让Nginx拒绝非法的请求方法：  

```shell  
http{  
   server {  
             listen 80;  
#这里，!符号表示对正则取反，~符号是正则匹配符号  
#如果用户使用非GET或POST方法访问网站，则retrun返回错误信息  
          if ($request_method !~ ^(GET|POST)$ ) {  
                 return 444;  
           }      
    }  
}  
```  

测试  
curl -i -X GET  <http://192.168.4.7>            #正常  
curl -i -X HEAD <http://192.168.4.7>            #失败  
  
### 3. 防止buffer溢出  

当客户端连接服务器时，服务器会启用各种缓存，用来存放连接的状态信息。  
如果攻击者发送大量的连接请求，而服务器不对缓存做限制的话，内存数据就有可能溢出（空间不足）。  
修改Nginx配置文件，调整各种buffer参数，可以有效降低溢出风险。  

```shell  
http{  
client_body_buffer_size  1k;  
client_header_buffer_size 1k;  
client_max_body_size 1k;  
large_client_header_buffers 2 1k;  
 … …  
}  
```  
  
### 4. 限制并发量  

DDOS攻击者会发送大量的并发连接，占用服务器资源（包括连接数、带宽等），这样会导致正常用户处于等待或无法访问服务器的状态。  
Nginx提供了一个ngx_http_limit_req_module模块，可以有效降低DDOS攻击的风险，操作方法如下：  

```shell  
http{  
… …  
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;  
server {  
    listen 80;  
    server_name localhost;  
    limit_req zone=one burst=5;  
        }  
}  
```  

> 备注说明：  
> limit_req_zone语法格式如下：  
> limit_req_zone key zone=name:size rate=rate;  
> 上面案例中是将客户端IP信息存储名称为one的共享内存，内存空间为10M  
> 1M可以存储8千个IP信息，10M可以存储8万个主机连接的状态，容量可以根据需要任意调整  
> 每秒中仅接受1个请求，多余的放入漏斗  
> 漏斗超过5个则报错  
  
测试

```shell  
ab -c 100 -n 100  http://192.168.4.7/  
```  

启用配置前测试结果  

```shell  
Server Software:        nginx  
Server Hostname:        192.168.4.7  
Server Port:            80  
  
Document Path:          /  
Document Length:        612 bytes  
  
Concurrency Level:      100  
Time taken for tests:   0.011 seconds  
Complete requests:      100  
Failed requests:        0  
Write errors:           0  
Total transferred:      83800 bytes  
HTML transferred:       61200 bytes  
Requests per second:    9331.84 [#/sec] (mean)  
Time per request:       10.716 [ms] (mean)  
Time per request:       0.107 [ms] (mean, across all concurrent requests)  
Transfer rate:          7636.80 [Kbytes/sec] received  
  
Connection Times (ms)  
              min  mean[+/-sd] median   max  
Connect:        0    3   1.0      4       5  
Processing:     2    2   0.5      2       7  
Waiting:        0    2   0.4      2       3  
Total:          5    6   0.8      6       7  
WARNING: The median and mean for the initial connection time are not within a normal deviation  
        These results are probably not that reliable.  
  
Percentage of the requests served within a certain time (ms)  
  50%      6  
  66%      6  
  75%      6  
  80%      6  
  90%      7  
  95%      7  
  98%      7  
  99%      7  
 100%      7 (longest request)  
```  
  
启用配置后结果  

```shell  
Server Software:        nginx  
Server Hostname:        192.168.4.7  
Server Port:            80  
  
Document Path:          /  
Document Length:        612 bytes  
  
Concurrency Level:      100  
Time taken for tests:   5.002 seconds  
Complete requests:      100  
Failed requests:        94  
   (Connect: 0, Receive: 0, Length: 94, Exceptions: 0)  
Write errors:           0  
Non-2xx responses:      94  
Total transferred:      69042 bytes  
HTML transferred:       50108 bytes  
Requests per second:    19.99 [#/sec] (mean)  
Time per request:       5001.735 [ms] (mean)  
Time per request:       50.017 [ms] (mean, across all concurrent requests)  
Transfer rate:          13.48 [Kbytes/sec] received  
  
Connection Times (ms)  
              min  mean[+/-sd] median   max  
Connect:        0    3   0.9      3       5  
Processing:     2  152 728.8      2    4996  
Waiting:        0  151 728.8      2    4996  
Total:          4  155 729.0      5    5000  
  
Percentage of the requests served within a certain time (ms)  
  50%      5  
  66%      6  
  75%      6  
  80%      6  
  90%      6  
  95%   1003  
  98%   4000  
  99%   5000  
 100%   5000 (longest request)  
```  

### 5. 更改 Nginx 服务的默认用户  

就像更改 ssh 的默认 22 端口一样，增加安全性，Nginx 服务的默认用户是 nobody ，我们更改为 nginx  

1) 创建 nginx 用户  
useradd -s/sbin/nologin -M  nginx  
-s表示指定用户所用的shell，此处为/sbin/nologin，表示不登录。  
-M表示不创建用户主目录。  
-g表示指定用户的组名为mysql。  
2) 更改配置文件  

```shell  
worker_processes 1;  
user nginx nginx;          # 指定Nginx服务的用户和用户组  
events {  
}  
http {  
}  
```  

3) 重新加载配置文件  

```shell
sbin/nginx -t  
sbin/nginx -s reload  
```  

4) 验证是否生效  

```shell
ps aux | grep nginx   
```  

### 6. 更改端口号  

修改配置文件(nginx.conf)  

```shell
http{  
server {  
        listen   10000;  # 默认端口80  
}  
```  
  
## 性能优化  

### 1. 优化 Nginx worker 进程数  

查看 cpu 核数  
Nginx 有 Master 和 worker 两种进程，Master 进程用于管理 worker 进程，worker 进程用于 Nginx 服务  
  
worker 进程数应该设置为等于 CPU 的核数，高流量并发场合也可以考虑将进程数提高至 CPU 核数 * 2  

```shell  
grep -c processor /proc/cpuinfo  
```  

更改 worker 进程数 nginx 配置文件(nginx.conf)  

```shell  
worker_processes 6;  
user nginx nginx;  
```  

重新加载配置  

```shell  
sbin/nginx -t  
sbin/nginx -s reload  
```  

检验是否修改成功  

```shell  
ps -ef | grep nginx | grep -v grep  
root     487215  0.0  0.0  56516  2224 ?        Ss   14:26   0:00 nginx: master process sbin/nginx  
nobody   626145  0.0  0.0  56944  2064 ?        S    16:25   0:00 nginx: worker process  
nobody   626146  0.0  0.0  56944  2064 ?        S    16:25   0:00 nginx: worker process  
nobody   626147  0.0  0.0  56944  2064 ?        S    16:25   0:00 nginx: worker process  
nobody   626148  0.0  0.0  56944  2064 ?        S    16:25   0:00 nginx: worker process  
nobody   626149  0.0  0.0  56944  2064 ?        S    16:25   0:00 nginx: worker process  
nobody   626150  0.0  0.0  56944  2064 ?        S    16:25   0:00 nginx: worker process  
```  

### 2. 绑定 nginx 进程到不同 cpu 也可自动绑定  

修改配置文件（nginx.conf）  

```  
worker_processes 2;        # 2核CPU的配置  
worker_cpu_affinity01 10;  
   
worker_processes 4;        # 4核CPU的配置  
worker_cpu_affinity0001 0010 0100 1000;     
   
worker_processes 8;        # 8核CPU的配置  
worker_cpu_affinity00000001 00000010 00000100 00001000 00010000 00100000 01000000 1000000;  
```  
  
## 2. Nginx 处理事件模型  

Nginx 的连接处理机制在不同的操作系统会采用不同的 I/O 模型，要根据不同的系统选择不同的事件处理模型，可供选择的事件处理模型有：kqueue 、rtsig 、epoll 、/dev/poll 、select 、poll ，其中 select 和 epoll 都是标准的工作模型，kqueue 和 epoll 是高效的工作模型，不同的是 epoll 用在 Linux 平台上，而 kqueue 用在 BSD 系统中。  
  
(1) 在 Linux 下，Nginx 使用 epoll 的 I/O 多路复用模型  
(2) 在 Freebsd 下，Nginx 使用 kqueue 的 I/O 多路复用模型  
(3) 在 Solaris 下，Nginx 使用 /dev/poll 方式的 I/O 多路复用模型  
(4) 在 Windows 下，Nginx 使用 icop 的 I/O 多路复用模型  
  
```  
events {  
    use epoll;  
}  
```  

### 3. 优化 Nginx 单个进程允许的最大连接数  

 (1) 控制 Nginx 单个进程允许的最大连接数的参数为 worker_connections ，这个参数要根据服务器性能和内存使用量来调整  
 (2) 进程的最大连接数受 Linux 系统进程的最大打开文件数限制，只有执行了 "ulimit -HSn 65535" 之后，worker_connections 才能生效。临时生效重启后无效  
 (3) 连接数包括代理服务器的连接、客户端的连接等，Nginx 总并发连接数 = worker 数量 * worker_connections, 总数保持在3w左右  
  
终极解除 Linux 系统的最大进程数和最大文件打开数限制：  
vim /etc/security/limits.conf  
\# 添加如下的行  

```  
* soft nproc 11000  
* hard nproc 11000  
* soft nofile 655350  
* hard nofile 655350  
```

修改nginx 配置文件  

```  
worker_processes 6;  
worker_cpu_affinity auto;  
user nginx nginx;  
events {  
    use epoll;  
    worker_connections 15000;  
}  
 ```  

### 4. 优化 Nginx worker 进程最大打开文件数  

```  
worker_processes 6;  
worker_cpu_affinity auto;  
user nginx nginx;  
worker_rlimit_nofile 65535;    # worker 进程最大打开文件数，可设置为优化后的 ulimit -HSn 的结果  
```  

### 5. 优化服务器域名的散列表大小  
  
### 6. 开启高效文件传输模式  

(1) sendfile 参数用于开启文件的高效传输模式，该参数实际上是激活了 sendfile() 功能，sendfile() 是作用于两个文件描述符之间的数据拷贝函数，这个拷贝操作是在内核之中的，被称为 "零拷贝" ，sendfile() 比 read 和 write 函数要高效得多，因为 read 和 write 函数要把数据拷贝到应用层再进行操作  
  
(2) tcp_nopush 参数用于激活 Linux 上的 TCP_CORK socket 选项，此选项仅仅当开启 sendfile 时才生效，tcp_nopush 参数可以允许把 http response header 和文件的开始部分放在一个文件里发布，以减少网络报文段的数量  

```  
http {  
include       mime.types;  
server_names_hash_bucket_size 512;      
default_type  application/octet-stream;  
sendfile      on;   # 开启文件的高效传输模式  
tcp_nopush    on;   # 激活 TCP_CORK socket 选择  
    tcp_nodelay on; #数据在传输的过程中不进缓存  
keepalive_timeout 65;  
server_tokens off;  
include vhosts/*.conf;  
}  
```

### 7. 优化 Nginx 连接超时时间  

连接超时的作用  
(1) 将无用的连接设置为尽快超时，可以保护服务器的系统资源（CPU、内存、磁盘）  
(2) 当连接很多时，及时断掉那些建立好的但又长时间不做事的连接，以减少其占用的服务器资源  
(3) 如果黑客攻击，会不断地和服务器建立连接，因此设置连接超时以防止大量消耗服务器的资源  
(4) 如果用户请求了动态服务，则 Nginx 就会建立连接，请求 FastCGI 服务以及后端 MySQL 服务，设置连接超时，使得在用户容忍的时间内返回数据  
  
连接超时存在的问题  
(1) 服务器建立新连接是要消耗资源的，因此，连接超时时间不宜设置得太短，否则会造成并发很大，导致服务器瞬间无法响应用户的请求  
(2) 有些 PHP 站点会希望设置成短连接，因为 PHP 程序建立连接消耗的资源和时间相对要少些  
(3) 有些 Java 站点会希望设置成长连接，因为 Java 程序建立连接消耗的资源和时间要多一些，这时由语言的运行机制决定的  
  
设置连接超时  
(1) keepalive_timeout ：该参数用于设置客户端连接保持会话的超时时间，超过这个时间服务器会关闭该连接  
(2) client_header_timeout ：该参数用于设置读取客户端请求头数据的超时时间，如果超时客户端还没有发送完整的 header 数据，服务器将返回 "Request time out (408)" 错误  
(3) client_body_timeout ：该参数用于设置读取客户端请求主体数据的超时时间，如果超时客户端还没有发送完整的主体数据，服务器将返回 "Request time out (408)" 错误  
(4) send_timeout ：用于指定响应客户端的超时时间，如果超过这个时间，客户端没有任何活动，Nginx 将会关闭连接  
(5) tcp_nodelay ：默认情况下当数据发送时，内核并不会马上发送，可能会等待更多的字节组成一个数据包，这样可以提高 I/O 性能，但是，在每次只发送很少字节的业务场景中，使用 tcp_nodelay 功能，等待时间会比较长  

```
http {
    include       mime.types;
    server_names_hash_bucket_size 512;    
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout 65;
    tcp_nodelay on;
    client_header_timeout 15;
    client_body_timeout 15;
    send_timeout 25;
    include vhosts/*.conf;
}
```

### 8. 限制上传文件的大小

 client_max_body_size 用于设置最大的允许客户端请求主体的大小，在请求首部中有 "Content-Length" ，如果超过了此配置项，客户端会收到 413 错误，即请求的条目过大

```

http {
    include       mime.types;
    server_names_hash_bucket_size 512;    
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout 65;
    server_tokens off;
    client_max_body_size8m;   # 设置客户端最大的请求主体大小为8M
    include vhosts/*.conf;
}
```

## 压缩优化

Nginx gzip 压缩模块提供了压缩文件内容的功能，用户请求的内容在发送到客户端之前，Nginx 服务器会根据一些具体的策略实施压缩，以节约网站出口带宽，同时加快数据传输效率，来提升用户访问体验，需要压缩的对象有 html 、js 、css 、xml 、shtml ，图片和视频尽量不要压缩，因为这些文件大多都是已经压缩过的，如果再压缩可能反而变大，另外，压缩的对象必须大于 1KB，由于压缩算法的特殊原因，极小的文件压缩后可能反而变大

```
http {
    gzip  on;                   # 开启压缩功能
    gzip_min_length 1k;        # 允许压缩的对象的最小字节
    gzip_buffers 4 32k;        # 压缩缓冲区大小，表示申请4个单位为16k的内存作为压缩结果的缓存
    gzip_http_version 1.1;     # 压缩版本，用于设置识别HTTP协议版本
    gzip_comp_level 9;         # 压缩级别，1级压缩比最小但处理速度最快，9级压缩比最高但处理速度最慢
    gzip_types text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml text/javascript application/javascript application/x-javascript text/x-json application/json application/x-web-app-manifest+json text/css text/plain text/x-component font/opentype application/x-font-ttf application/vnd.ms-fontobject image/x-icon;        # 允许压缩的媒体类型
    gzip_vary  on;              # 该选项可以让前端的缓存服务器缓存经过gzip压缩的页面，例如用代理服务器缓存经过Nginx压缩的数据
}
```

## 日志优化

## 防爬虫

nginx/conf 目录下创建 agent-deny.conf 文件
添加以下内容

```
if ($http_user_agent ~* (Scrapy|Curl|HttpClient)) {
     return 403;
}
 
#禁止指定UA及UA为空的访问
if ($http_user_agent ~ "WinHttp|WebZIP|FetchURL|node-superagent|java/|
FeedDemon|Jullo|JikeSpider|Indy Library|Alexa Toolbar|AskTbFXTV|AhrefsBot|
CrawlDaddy|Java|Feedly|Apache-HttpAsyncClient|UniversalFeedParser|ApacheBench|
Microsoft URL Control|Swiftbot|ZmEu|oBot|jaunty|Python-urllib|
lightDeckReports Bot|YYSpider|DigExt|HttpClient|MJ12bot|heritrix|EasouSpider|Ezooms|BOT/0.1|
YandexBot|FlightDeckReports|Linguee Bot|^$" ) {
     return 403;             
}
```

## 禁止 ip 访问, 仅域名访问可用

```
server {

        listen 443 ssl;


        ssl_certificate  cert/_.cscec.com_bundle.crt;
        ssl_certificate_key cert/_.cscec.com.key;
        server_name _;
        return 403;
    }
```

[文章连接](https://blog.csdn.net/IT_Most/article/details/108784791)
