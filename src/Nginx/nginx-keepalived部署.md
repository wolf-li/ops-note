# 安装keepalived ##


## 通过离线包进行安装：
注：主节点比备份节点 优先级高
1. 进入rpm包目录：
`rpm -ivh *.rpm --force --nodeps `

2. 创建校验脚本（master/backup 主机都要创建）
/etc/keepalived/check_nginx.sh

> A=`ps -C nginx --no-header |wc -l`
> if [ $A -eq 0 ];then
>         /data/nginx/sbin/nginx
>         sleep 2
>         if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
>                 killall keepalived
>         fi
> fi

3. 修改master1 配置文件
vim /etc/keepalived/keepalived.conf

> global_defs {
>    router_id ecs-b52zu30sgqy4  #主机名
> }
> vrrp_script check_nginx {
>    script "/etc/keepalived/check_nginx.sh"
>    interval 2
>    weight -20
> }
> 
> vrrp_instance VI_1 {
>     state MASTER
>     interface eth0   #网口名
>     virtual_router_id 53
>     unicast_src_ip  10.250.60.42 
>     unicast_peer {
>         10.250.60.43
>     }
>     priority 110     #优先级
>     advert_int 1
>     authentication {
>         auth_type PASS
>         auth_pass 1111
>     }
>     track_script {
>        check_nginx
>     }
>     virtual_ipaddress {
>         10.250.60.122   #VIP
>     }
> }

4. 启动keepalived进程
```
sudo systemctl start keepalived
sudo systemctl enable keepalived
```

5. backup主机配置文件配置
vim /etc/keepalived/keepalived.conf

> global_defs {
>    router_id i-FJNom8qrYH-5
> }
> 
> vrrp_script chk_nginx {
>    script "/etc/keepalived/check_nginx.sh"
>    interval 2
>    weight -20
> }
> 
> vrrp_instance VI_1 {
>     state BACKUP
>     interface eth0
>     virtual_router_id 53
>     unicast_src_ip  10.250.60.43
>     unicast_peer {
>         10.250.60.42
>     }
>     priority 100
>     nopreempt
>     advert_int 1
>     authentication {
>        auth_type PASS
>        auth_pass 1111
>     }
>     track_script {
>        chk_nginx
>     }
>     virtual_ipaddress {
>        10.250.60.122
>     }
> }

6. 启动keepalived进程
```
sudo systemctl start keepalived
sudo systemctl enable keepalived
```

## keepalived 配置日志

```shell
sed -i "s/KEEPALIVED_OPTIONS=\"-D\"/KEEPALIVED_OPTIONS=\"-D -d -S 0\"/g" /etc/sysconfig/keepalived
sed -i "/RSYSLOG_TraditionalFileFormat/a\local0.* \/var\/log\/keepalived.log" /etc/rsyslog.conf
systemctl restart rsyslog
systemctl restart keepalived
if [[ -f /var/log/keepalived.log ]] ; then
   echo "keepalived log file create"
else
   echo "creating keepalived log file fail"
fi
```
---
参考文章
https://blog.csdn.net/nangeali/article/details/81604432