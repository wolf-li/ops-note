

## 启动失败
1. 查看日志
tail -10 /var/log/mysqld.log


my_default.cnf

./bin/mysqld --initialize --user=mysql --basedir=/data/mysql/ --datadir=/data/mysql/data/

zhongjianmenhu


## 数据库连接

> mysql -h mysql服务器的IP地址 -P 端口号（通常为3306） -u 用户名 -p密码       
> 
> -h: mysql服务器的IP地址
> -P: 大写的P选项表示端口号，端口号默认为3306，可省略
> -u: 用户名
> -p: 小写的p表示密码，当-p后输入密码时，会直接登陆。当-p后不输入密码时，会要求输入密码，但密码不显示

```shell
mysql -h 192.168.5.116 -P 3306 -u root -p123456   //显示密码登陆
```

## 版本查看
bin安装文件查看
/data/mysql/docs/INFO_SRC  




