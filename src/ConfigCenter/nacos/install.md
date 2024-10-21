# Nacos 
Nacos /nɑ:kəʊs/ 是 Dynamic Naming and Configuration Service的首字母简称，一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。


## install 
1.预备环境准备
Nacos 依赖 Java 环境来运行。如果您是从代码开始构建并运行Nacos，还需要为此配置 Maven环境，请确保是在以下版本环境中安装使用:

64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
64 bit JDK 1.8+；
Maven 3.2.x+  注释：若是下载 nacos 的安装包不许要这个

```
tar -xvf nacos-server-$version.tar.gz
cd nacos/bin
sh startup.sh -m standalone   # 单机模式启动

sh shutdown.sh  # 关闭服务
```

默认 8848 端口
修改位置配置文件 application.properties 单机时
浏览器登录控制台默认账号： nacos/nacos


---
参考文章：
https://nacos.io/zh-cn/docs/quick-start.html





