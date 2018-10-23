---
layout: post
title: redis进阶(3)
date: 2018-07-30 10:20:57
tags: redis
categories: redis
---

### Redis序列化

Redis 协议将传输的结构数据分为 5 种最小单元类型，单元结束时统一加上回车换行符号`\r\n`。

- 单行字符串 以` + ` 符号开头。
- 多行字符串 以` $ ` 符号开头，后跟*字符串长度*。
- 整数值 以 ` : `符号开头，后跟*整数*的字符串形式。
- 错误消息 以` - `符号开头。
- 数组 以 ` * `号开头，后跟*数组的长度*。

>NULL 用多行字符串表示，不过长度要写成-1，`$-1\r\n`。空串 用多行字符串表示，长度填 0，`$0\r\n\r\n`。

例子：一个简单的 set 指令`set author codehole`会被序列化成下面的字符串:
`*3\r\n$3\r\nset\r\n$6\r\nauthor\r\n$8\r\ncodehole\r\n`,在控制台打印：
```shell
*3
$3
set
$6
author
$8
codehole
```
<!-- more -->

返回结果：`OK`其实是`+OK`(单行字符串响应)

>注意：客户端向服务器发送的指令只有一种格式，多行字符串数组。而服务器向客户端发送的指令才是上面五钟类型的组合。


### Redis持久化
两种方式：
- RDB 持久化，可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot），自动保存到系统硬盘上。
- AOF 持久化，记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。

通常 Redis 的主节点是不会进行持久化操作，持久化操作主要在从节点进行。从节点是备份节点，没有来自客户端请求的压力，它的操作系统资源往往比较充沛。另外还应该增加多个从节点以降低网络分区的概率，只要有一个从节点数据同步正常，数据也就不会轻易丢失。

### Redis发布者订阅者模型
在两个客户端之间完成消息的订阅和发布。

客户端1（订阅）：
```shell
> subscribe codehole.image codehole.text codehole.blog  # 同时订阅三个主题，会有三条订阅成功反馈信息
1) "subscribe"
2) "codehole.image"
3) (integer) 1
1) "subscribe"
2) "codehole.text"
3) (integer) 2
1) "subscribe"
2) "codehole.blog"
3) (integer) 3
```

客户端2（发布）：
```shell
> publish codehole.image https://www.google.com/dudo.png
(integer) 1
> publish codehole.text " 你好，欢迎加入码洞 "
(integer) 1
> publish codehole.blog '{"content": "hello, everyone", "title": "welcome"}'
(integer) 1
```
消息发布完成后，可以在客户端1上查看订阅的消息。
当然，redis也提供了匹配模式的订阅方式：
```shell
> psubscribe codehole.*  # 用模式匹配一次订阅多个主题，主题以 codehole. 字符开头的消息都可以收到
```

### Redis  5.0 新特性Stream

### LRU淘汰策略

Redis中提供了6种数据淘汰策略：

- `noeviction`: 禁止驱逐数据,默认方式。
- `allkeys-lru`: 当内存不足，从数据集中挑选最近最少使用（LRU算法）的数据淘汰。
- `allkeys-random`: 从数据集中任意随机选择数据淘汰。
- `volatile-lru`: 当内存不足，从已设置过期时间的数据集中挑选最近最少使用（LRU算法）的数据淘汰
- `volatile-random`: 从已设置过期时间的数据集中任意随机选择数据淘汰
- `volatile-ttl`: 从已设置过期时间的数据集中挑选将要过期的数据（TTL存活时间最短的）淘汰

`volatile-xxx` 策略只会针对带过期时间的 key 进行淘汰，`allkeys-xxx` 策略会对所有的 key 进行淘汰。如果你只是拿 Redis 做缓存，那应该使用 allkeys-xxx，客户端写缓存时不必携带过期时间。如果你还想同时使用 Redis 的持久化功能，那就使用 volatile-xxx 策略，这样可以保留没有设置过期时间的 key，它们是永久的 key 不会被 LRU 算法淘汰。

参考：http://antirez.com/latest/0



