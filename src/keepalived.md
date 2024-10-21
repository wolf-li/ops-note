v2.0 版本格式

master
```
# master 配置
global_defs {
   script_user root
   enable_script_security
}
 
vrrp_script check_script {
   script "/etc/keepalived/check_redis.sh"
   interval 2
   weight -20
}
 
vrrp_instance VI_1 {
    state MASTER 
    priority 100    # 节点的优先级，可选值为 [1-255]
    interface eth0  # vip绑定的网口名
    virtual_router_id 51  # 设置VRID标记，取值在0-255之间。同一集群内virtual_router_id保持一致。
    advert_int 1
    unicast_src_ip 10.250.60.42
    unicast_peer {
        10.250.60.43
    }
    authentication {
        auth_type PASS
        auth_pass 1111  # 验证密码,默认1111
    }
    virtual_ipaddress {
        10.240.127.107   # VIP
    }
    track_script {
        check_script
    }
}
```

```
# backup 配置
vrrp_instance VI_1 {
    state BACKUP
    nopreempt
    interface eth0
    virtual_router_id 51
    priority 90
    advert_int 1
    unicast_src_ip 10.250.60.42
    unicast_peer {
        10.250.60.43
    }
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.240.127.107
    }
}

```
脚本权限 754
/etc/keepalived/check_redis.sh

https://wiki.archlinux.org/title/Keepalived