---
layout: post
title: redis--事务处理
date: 2018-08-06 03:26:09
tags: redis
categories: redis
---

### 通过MULTI开启事物，然后命令入列，EXEC结束事物
1、命令行操作：
```shell
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET Book_Name "GIt Pro"
QUEUED
127.0.0.1:6379> SADD Program_Language "C++" "C#" "Jave" "Python" 
QUEUED
127.0.0.1:6379> GET Book_Name
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) (integer) 4
3) "GIt Pro"
```

2、java代码：

```java
redisTemplate.multi();
redisTemplate.lpush("key", "11");
redisTemplate.lpush("key", "22");  
redisTemplate.lpush("key", "33");
redisTemplate.exec();
```
注意一定要在配置文件中，开启事物：

<!-- more -->

```java
RedisTemplate<String, Object> template = new RedisTemplate<>();
/** 开启事务*/
template.setEnableTransactionSupport(true);
```

>如果客户端在使用 `MULTI` 开启了一个事务之后，却因为断线而没有成功执行 `EXEC` ，则Redis会清空事务队列，那么事务中的所有命令都不会被执行。
另一方面，如果客户端成功在开启事务之后执行 `EXEC` ，那么事务中的所有命令都会被执行。

>通过调用 `DISCARD` ， 客户端可以清空事务队列， 并放弃执行事务。

最后同样重要的是：
1.如果 Redis 事务里的一条命令出现了语法错误，不执行任何操作，直接 `discard` 退出事物。
2.如果 Redis 事务里的一条命令出现了运行错误，事务里其他的命令依然会继续执行（包括出错命令之后的命令）。

所以 redis 中的事物不满足原子性，只满足隔离性，redis 是以更简单、更快速的无回滚方式来处理事务的。出现运行时错误时，要想实现回滚，只能通过手动操作来恢复之前的状态。


### WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为（实现乐观锁，避免竞争和碰撞）。
可处理的情况：有些情况下需要先获得一条命令的返回值，然后再根据这个值执行下一条命令。

1.`WATCH` 使得 `EXEC` 命令需要有条件地执行： 事务只能在所有被监视键都没有被修改的前提下执行， 如果这个前提不能满足的话，事务就不会被执行。
2.如果你使用 `WATCH` 监视了一个带过期时间的键， 那么即使这个键过期了， 事务仍然可以正常执行。
3.`WATCH` 命令可以被调用多次。 对键的监视从 `WATCH` 执行之后开始生效， 直到调用 `EXEC` 为止，不管事物成功与否。当然客户端断开连接,监视也会被取消。
4.用户还可以在单个 `WATCH` 命令中监视任意多个键。
5.使用无参数的 `UNWATCH` 命令可以手动取消对所有键的监视。

