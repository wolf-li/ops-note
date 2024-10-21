1. 查看原有 nginx 配置信息
```
nginx -V
nginx version: nginx/1.20.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/data/nginx --with-http_stub_status_module --with-http_ssl_module
```

2. 进入 nginx 源码文件，重新编译
cd /data/nginx-1.20.1
./configure --prefix=/data/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre --add-module=/data/ngx_cache_purge-2.5.1
make && make install
make upgrade

注意的点：

1. configure --prefix=/data/nginx --with-http_stub_status_module --with-http_ssl_module

重新编译时记得 configure 保持一致后在添加第三方模块

2. 有 keepalived 存在情况下，这种升级方案也是可用的可以做到热升级。
