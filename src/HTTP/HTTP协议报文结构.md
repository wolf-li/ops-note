# HTTP 协议报文结构

## HTTP 请求报文

浏览器给 Web 服务器的 HTTP 请求报文内容

```bash
起始行： POST http://tankxiao.vi/user-login.html HTTP/1.1
首部：   HOST: tankxiao.vi
        User-Agent: Mozila/5.0

主体:    account=book&password=2131242534
```

请求报文详细内容

```bash
Header 首部  
POST http://tankxiao.io/user-login.html  HTTP/1.1
HOST: tankxiao.io
User-Agent: Mozila/5.0
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Refer: http://tankxiao.io/user-login.html
Connection: keep-alive

Body 主体
account=book&password=2131242534
```
Header首部和Body 主体之间有一个空行

## HTTP 响应报文

HTTP 响应报文内容
起始行  HTTP/1.1 200 OK
首部    Content-Length: 123
       Content-type: text/html

主体    <html>
        <head><title>小花园</title>
        </head>
        </html>

第一行为起始行，有状态码消息
第二行首部（header）
第三部分（Body）