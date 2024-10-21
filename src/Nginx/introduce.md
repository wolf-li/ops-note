<!--
 * @Author: wolf-li
 * @Date: 2024-10-20 19:47:49
 * @LastEditTime: 2024-10-20 19:57:55
 * @LastEditors: wolf-li
 * @Description: 
 * @FilePath: /note/src/Nginx/introduce.md
 * talk is cheep show me your code.
-->
# Nginx
nginx 是一个 HTTP Web 服务，反向代理，内容缓存，负载均衡，TCP/UDP 代理服务，邮件代理服。
[官方网址](https://nginx.org)
## 解决问题
* 高并发：Nginx 能够处理大量的并发连接，适用于高流量的网站
* 高性能：Nginx 性能优异，可以提供快速响应
* 稳定性：Nginx 非常稳定，很少出现崩溃
* 灵活配置：Nginx 的配置灵活，可以满足各种复杂的应用场景。
  
## 核心原理
Nginx 的工作原理主要基于以下几个方面：
• 异步非阻塞： Nginx 采用异步非阻塞的方式处理请求，一个 worker 进程可以同时处理多个请求，提高了并发处理能力。
• 事件驱动： Nginx 基于事件驱动模型，当有事件发生（如客户端连接、数据接收等）时，就会触发相应的事件处理器。
• master-worker 模式： Nginx 采用 master-worker 模式，master 进程负责管理 worker 进程，worker 进程负责处理实际的请求。
模块化设计： Nginx 的功能通过模块实现，可以灵活地添加或删除模块。

## 部署方式
Nginx 的部署方式主要有以下几种：  
* 源码编译安装： 灵活性高，可以自定义配置，但需要一定的编译知识。 
* 包管理器安装： 方便快捷，适用于大多数 Linux 发行版。
* 容器化部署： 使用 Docker 等容器技术，方便部署和管理。

## 组成部分
Nginx 的核心组件包括：
• master 进程： 管理 worker 进程，监控配置，重新加载配置文件等。
• worker 进程： 处理网络事件，完成请求的处理。
• 事件模块： 处理各种 I/O 事件。
• HTTP 模块： 处理 HTTP 请求。
• 邮件模块： 处理邮件相关的协议。

## 应用场景
Nginx 在以下场景中应用广泛：
* 静态文件服务器： 提供高性能的静态文件服务。
* 反向代理： 将客户端请求转发到后端服务器集群。
* 负载均衡： 将负载分发到多个后端服务器。
* HTTP 缓存： 缓存静态资源，减少后端服务器的压力。

