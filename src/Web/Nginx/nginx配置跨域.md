

#允许跨域请求的域，* 代表所有
add_header 'Access-Control-Allow-Origin' *;
#允许请求的header
add_header 'Access-Control-Allow-Headers' *;
#允许带上cookie请求
add_header 'Access-Control-Allow-Credentials' 'true';
#允许请求的方法，比如 GET,POST,PUT,DELETE
add_header 'Access-Control-Allow-Methods' *;
若考虑到安全性，也可以指定访问来源请求的域，示例：
add_header 'Access-Control-Allow-Origin' 'http://test1.xqiangme.top';