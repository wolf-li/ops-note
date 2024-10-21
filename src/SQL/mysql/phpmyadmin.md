# phpmyadmin
phpMyAdmin是一个以PHP为基础，以Web-Base方式架构在网站主机上的MySQL的数据库管理工具，让管理者可用Web接口管理MySQL数据库。借由此Web接口可以成为一个简易方式输入繁杂SQL语法的较佳途径，尤其要处理大量资料的汇入及汇出更为方便。其中一个更大的优势在于由于phpMyAdmin跟其他PHP程式一样在网页服务器上执行，但是您可以在任何地方使用这些程式产生的HTML页面，也就是于远端管理MySQL数据库，方便的建立、修改、删除数据库及资料表。也可借由phpMyAdmin建立常用的php语法，方便编写网页时所需要的sql语法正确性。

## 部署
### 1）docker 容器部署


phpmyadmin 容器部署
```
# 拉取镜像
docker pull phpmyadmin
# 启动镜像
# PMA_HOST变量，指 mysql服务器地址
docker run --name phpmyadmin -d \
    -e PMA_HOST=172.17.0.5 \
    -p 8080:80 \
    phpmyadmin/phpmyadmin
```

phpmyadmin 连接单独部署 mysql
```bash
docker run --name myadmin -d \
-e PMA_HOSTS=10.6.7.27,10.6.7.11 \
-e PMA_PORTS=3406,3306  \
-p 8080:80 \
-v /data/phpmyadmin/config.user.inc.php:/etc/phpmyadmin/config.user.inc.php\
phpmyadmin
```
[phpmyadmin docker 官网地址](https://hub.docker.com/_/phpmyadmin)
[phpmyadmin nginx 反向代理](https://blog.csdn.net/weixin_46396833/article/details/123381270)

---
参考文档：
https://www.cnblogs.com/haoprogrammer/p/11008786.html#:~:text=docker%E9%83%A8%E7%BD%B2mysql%3A5.7.26%20%23%20%E4%B8%8B%E8%BD%BD%E9%95%9C%E5%83%8F%20docker%20pull%20mysql%3A5.7.%2026%20%23,-v%20%24PWD%2Fconf%3A%2Fetc%2Fmysql%20-v%20%24PWD%2Fdata%3A%2Fvar%2Flib%2Fmysql%20-e%20MYSQL_ROOT_PASSWORD%3D123456%20-d%20mysql%3A5.7.26
