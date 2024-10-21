# 根据版本进行相应修改



# 临时修复
runbroker.sh
添加代码
```
JAVA_OPT="${JAVA_OPT}  -Dfastjson.parser.safeMode=true"
```

## 测试
poc 测试






参考文章：
https://github.com/alibaba/fastjson/wiki/security_update_20220523
https://nosec.org/home/detail/5005.html

1. 升级 fastjson 最新版本 或 fastjson2
2. fastjson 开启 safemode 
https://github.com/alibaba/fastjson/wiki/fastjson_safemode
