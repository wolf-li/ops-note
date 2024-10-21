# redis 简介

* 速度快（数据放入内存中）
* c 语言实现
* 使用单线程架构
* 丰富功能
  * 提供 key 过期功能，实现缓存
  * 提供发布订阅共功能
  * 支持 lua 脚本
  * 提供简单事物功能
  * 提供流水线功能，客户端可以将一批命令一次性传递到 redis，减少网络开销
* 简单稳定，代码量少，3版本5万
* 客户端语言多 Java 、python 、php、c 、c++ 、nodejs
* 持久化 （RDB|AOF） 将数据写入磁盘
* 主从复制
* 高可用分布式

## 使用场景

* 缓存
* 排行系统
* 计数器应用
* 社交网络
* 消息队列系统

## redis 编译后文件

名称|作用
--|--
redis-server| redis-server
redis-benchmark|used to check redis performances.
redis-cli|a CLI utility to interact with redis
redis-check-rdb|useful in the rare event of corrupted data files.
redis-check-aof|useful in the rare event of corrupted data files.
redis-sentinel|redis sentinel executable (monitoring and failover).

## redis API理解使用

### redis 全局命令

1. 查看所有健
`keys *`
2. 插入键值对

```
set hello world
set java jedis
set python redis-py
```

3. 插入一个 列表类型的键值对

```
rpush mylist a b c d e f g  #(mylist 健 值: a b c d e f g )
```

4. 显示当前 redis 有多少个健
`dbsize`
dbsize 与 keys 区别，
dbsize 计算健数量时不会遍历所有，而是直接获取 redis 内置的健总数量时间复杂度 O（1）
keys 会遍历所有健，时间复杂度 O（n），当redis 保存大量健时禁止使用此命令。
5. 查看某个健是否存在

```
> exists java
> (integer) 1
```

6. 删除健，无论值是什么数据类型

```
del key [ key .... ]
```

7. 设置健过期时间,当超过设置时间时，会自动删除健
`set [key] [second]`
8. 查看健的数据结构类型
`type [key]`
注：type 返回当前健的数据结构类型：分别是 string 、hash 、list、set 、zset，这些只是 redis 对外的数据结构。
实际上每种数据结构都有两种以上的内部，编码实现，如 list 结构包含了 linkedist 和 ziplist 两种内部编码。
9. 查看内部编码
`object encoding [key]`

### 单线程架构

所有从客户端传输的命令都会存储在一个队列里，然后逐个执行，
单线程优势：

* 单线程可以简化数据结构和算法的实现
* 单线程避免了线程切换，和竞态产生的消耗，对于服务端开发来说，锁和线程切换通常是性能杀手。
劣势：
* 对于每个命令执行时间是有要求的，如果命令执行时间过长，会造成其他命令阻塞，对于 redis 这种高性能服务来说是致命的。
