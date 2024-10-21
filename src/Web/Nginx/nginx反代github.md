server
{
    listen 443 ssl http2 reuseport;
    #http2 表示启用http2协议,http2一般需要openssl 1.0.2+以上版本支持,大部分linux系统需要自行编译nginx作为支持，ubuntu16.04可以直接支持。

    listen 80 reuseport;
    #同时监听80端口。
    #reuseport提高nginx性能，有兴趣可以搜索引擎

    location ~* .(conf|sql|bak)$ {
        deny all;
    }

    ssl_certificate ssl/p.run.la.pem;
    ssl_certificate_key ssl/p.run.la.key;

    ssl_certificate ssl/p.run.la.ecdsa.pem;
    ssl_certificate_key ssl/p.run.la.ecdsa.key;
    #配置双证书，一个是rsa算法的证书，一个是ecc,ecc算法效率更高，但是并不支持某些老的系统。

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    #配置加密协议，如果要支持ie6还需要加上SSLv3，但是安全性会降低

    ssl_session_timeout      1d;
    #缓存时间

    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+ECDSA+AES128:EECDH+aRSA+AES128:RSA+AES128:EECDH+ECDSA+AES256:EECDH+aRSA+AES256:RSA+AES256:EECDH+ECDSA+3DES:EECDH+aRSA+3DES:RSA+3DES:!MD5;
    #配置https加密算法,chacha20对称算法消耗更少，对于移动端更友好，但是需要自行编译nginx，打Cloudflare 补丁。

    ssl_prefer_server_ciphers on;


    ssl_session_cache        shared:SSL:50m;

    ssl_session_tickets      on;
    #配置ssl缓存

    ssl_stapling             on;
    #开启OCSP,因为Let's Encryptc证书的原因，我开启ocsp无需更多设置。

    server_name p.run.la; #绑定的域名

    if ($request_method !~ ^(GET|HEAD|POST)$ ) {
        return    444;
    }

    if ($http_user_agent ~* "qihoobot|Baiduspider|Googlebot|Googlebot-Mobile|Googlebot-Image|Mediapartners-Google|Adsbot-Google|Feedfetcher-Google|Yahoo! Slurp|Yahoo! Slurp China|YoudaoBot|Sosospider|Sogou spider|Sogou web spider|MSNBot|ia_archiver|Tomato Bot")
    {
        return 403;
    }
    #屏蔽搜索引擎

    resolver                 8.8.8.8 8.8.4.4  valid=600s;
    #设置域名解析的dns服务器
    resolver_timeout         10s;
    #设置dns解析超时时间。

    location ~ ^/(repo\.mongodb\.com|repo\.mysql\.com|www\.debian\.org|deb\.debian\.org|security\.debian\.org|cdn-fastly\.deb.debian\.org|nginx\.org|github\.com|codeload\.github\.com|yum\.dockerproject\.org)(\/.*)$ {
        proxy_http_version               1.1;
        proxy_connect_timeout            10s;
        proxy_read_timeout               10s;
        proxy_set_header Host            github.com;

        # 防止HSTS校验问题
        proxy_hide_header Strict-Transport-Security;
        
        # 上游就是GitHub
        proxy_pass https://github.com;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    root  /home/wwwroot/p.run.la;
    #设置目录。
}
