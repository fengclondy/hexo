---
layout: post
title: study--Redis
date: 2018-07-27 08:33:24
tags: study
categories: study
---


## Redis
### Redis单线程问题

单线程指的是网络请求模块使用了一个线程（所以不需考虑并发安全性），即一个线程处理所有网络请求，其他模块仍用了多个线程。

### 为什么说Redis能够快速执行

绝大部分请求是纯粹的内存操作（非常快速）

采用单线程,避免了不必要的上下文切换和竞争条件

非阻塞IO - IO多路复用

### Redis的内部实现

内部实现采用epoll，采用了epoll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，不在io上浪费一点时间 这3个条件不是相互独立的，特别是第一条，如果请求都是耗时的，采用单线程吞吐量及性能很差。redis为特殊的场景选择了合适的技术方案。

Redis关于线程安全问题

redis实际上是采用了线程封闭的观念，把任务封闭在一个线程，自然避免了线程安全问题，不过对于需要依赖多个redis操作的复合操作来说，依然需要锁，而且有可能是分布式锁。

### 使用Redis有哪些好处？

速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)

支持丰富数据类型，支持string，list，set，sorted set，hash

支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行

丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除

### redis相比memcached有哪些优势？

memcached所有的值均是简单的字符串，redis作为其替代者，支持更为丰富的数据类型

redis的速度比memcached快很多

redis可以持久化其数据

Redis支持数据的备份，即master-slave模式的数据备份。

使用底层模型不同，它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。

value大小：redis最大可以达到1GB，而memcache只有1MB

### Redis主从复制

过程原理：

当从库和主库建立MS关系后,会向主数据库发送SYNC命令

主库接收到SYNC命令后会开始在后台保存快照(RDB持久化过程),并将期间接收到的写命令缓存起来

当快照完成后,主Redis会将快照文件和所有缓存的写命令发送给从Redis

从Redis接收到后,会载入快照文件并且执行收到的缓存的命令

之后,主Redis每当接收到写命令时就会将命令发送从Redis，从而保证数据的一致

缺点：所有的slave节点数据的复制和同步都由master节点来处理,会照成master节点压力太大,使用主从从结构来解决

### Redis两种持久化方式的优缺点

- RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）
- AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。

Redis 还可以同时使用 AOF 持久化和 RDB 持久化。当redis重启时,它会有限使用AOF文件来还原数据集,因为AOF文件保存的数据集通常比RDB文件所保存的数据集更加完整

#### RDB的优点：

RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。 这种文件非常适合用于进行备份： 比如说，你可以在最近的 24 小时内，每小时备份一次 RDB 文件，并且在每个月的每一天，也备份一个 RDB 文件。 这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。

RDB 非常适用于灾难恢复（disaster recovery）：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心，或者亚马逊 S3 中。

RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。

RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快

### Redis常见的性能问题都有哪些？如何解决？

Master写内存快照，save命令调度rdbSave函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务，所以Master最好不要写内存快照。

Master AOF持久化，如果不重写AOF文件，这个持久化方式对性能的影响是最小的，但是AOF文件会不断增大，AOF文件过大会影响Master重启的恢复速度。Master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化,如果数据比较关键，某个Slave开启AOF备份数据，策略为每秒同步一次。

Master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂服务暂停现象。

Redis主从复制的性能问题，为了主从复制的速度和连接的稳定性，Slave和Master最好在同一个局域网内

### Redis提供6种数据淘汰策略

- `volatile-lru`：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
- `volatile-ttl`：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
- `volatile-random`：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
- `allkeys-lru`：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
- `allkeys-random`：从数据集（server.db[i].dict）中任意选择数据淘汰
- `no-enviction（驱逐）`：禁止驱逐数据