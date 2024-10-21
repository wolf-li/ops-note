# 清除缓存 ngx_cache_purge 模块
1. 下载模块 [github](https://github.com/midorinet/ngx_cache_purge)
2. nginx 添加模块
3. 修改 nginx 配置文件
```
location ~ /purge(/.*) {
    allow                     127.0.0.1;
    deny                      all;   # 限制只有 127.0.0.1 可以使用
    proxy_cache_purge         data_1  $host$1$is_args$args;
}
location / {
    proxy_cache data_1;
    proxy_cache_valid 200 304 2m; #对不同的HTTP状态码设置不同的缓存时间
    proxy_cache_key $host$uri$is_args$args;  # 这个要添加否则，无法通过 url 删除缓存
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_pass http://;  #反向代理请求地址
}
```
4. 删除所有 proxy_cache 缓存
```
http://127.0.0.1/purge/*
```

这里有个小问题
使用 url 连接删除前，手动清空目录。