# nginx 配置 https web应用

1．在服务器中安装OpenSSL工具包

通过如下两条命令安装OpenSSL：

```
sudo yum -y install openssl
sudo yum install openssl-devel
```

命令运行成功后，OpenSSL命令和配置文件将被安装到Linux系统目录中。

OpenSSL命令：/usr/bin/openssl。

配置文件：/usr/lib/ssl/*。

2．生成SSL密钥和证书

通过如下步骤生成CA证书ca.crt、服务器密钥文件server.key和服务器证书server.crt：

// 生成CA 密钥
 `openssl genrsa -out ca.key 2048`

// 生成CA证书，days参数以天为单位设置证书的有效期。在本过程中会要求输入证书的所在地、公司名、站点名等
`openssl req -x509 -new -nodes -key ca.key -days 365 -out ca.crt`

// 生成服务器证书RSA的密钥对
 `openssl genrsa -out server.key 2048`
// 生成服务器端证书CSR，本过程中会要求输入证书所在地、公司名、站点名等
`openssl req -new -key server.key -out server.csr`
// 生成服务器端证书 ca.crt
`openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365`
上述命令生成服务器端证书时，必须在Common Nanme（CN）字段中如实输入站点的访问地址。即如果站点通过www.mysite.com访问，则必须定义CN=www.mysite.com；如果通过IP地址访问，则需设置CN为具体的IP地址。
3．配置Nginx HTTPS服务器

nginx 需要开启下面的模块才能开启 https 服务
--with-http_stub_status_module --with-http_ssl_module

在站点配置文件/etc/nginx/sites-enabled/default中添加如下server段，可以定义一个基于HTTPS的接口，该接口的服务器端程序仍旧为uWSGI接口127.0.0.1:3011。

```
server {
listen       443 ssl;       # HTTPS服务端口
server_name 0.0.0.0;      # 本机上的所有IP地址

ssl_certificate  /etc/nginx/ssl/server.crt;
ssl_certificate_key  /etc/nginx/ssl/server.key;

    location \ {
        uwsgi_pass http://127.0.0.1:3011;
    }
}
```

其中需要注意的是参数ssl_certificate和ssl_certificate_key需要分别指定生成的服务器证书和服务器密钥的全路径文件名。
至此，我们已经可以使用浏览器访问服务器的443端口进行HTTPS加密通信了。

### 证书自签发脚本

```shell
#!/bin/sh

# create self-signed server certificate:

read -p "Enter your domain [www.example.com]: " DOMAIN

echo "Create server key..."

openssl genrsa -des3 -out $DOMAIN.key 1024

echo "Create server certificate signing request..."

SUBJECT="/C=US/ST=Mars/L=iTranswarp/O=iTranswarp/OU=iTranswarp/CN=$DOMAIN"

openssl req -new -subj $SUBJECT -key $DOMAIN.key -out $DOMAIN.csr

echo "Remove password..."

mv $DOMAIN.key $DOMAIN.origin.key
openssl rsa -in $DOMAIN.origin.key -out $DOMAIN.key

echo "Sign SSL certificate..."

openssl x509 -req -days 3650 -in $DOMAIN.csr -signkey $DOMAIN.key -out $DOMAIN.crt
```

相关文档：
<https://github.com/michaelliao/itranswarp.js/blob/master/conf/ssl/gencert.sh#L12>
<https://yunwei.blog.csdn.net/article/details/116303372?spm=1001.2014.3001.5502>
