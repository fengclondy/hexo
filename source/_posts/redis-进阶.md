---
layout: post
title: redis进阶
date: 2018-07-28 01:20:57
tags: redis
categories: redis
---


### 分布式锁
枷锁和释放锁的过程：
```shell
> set lock:codehole true ex 5 nx
OK
... do something critical ...
> del lock:codehole
```

### 延迟队列消息
异步队列使用list集合，命令 lpush 和 rpop 结合使用

但是无法保证ack机制，如果要保证可靠传输，redis做不到，可以使用rabbitmq或者Kafka

延迟队列，即阻塞读与写，用blpop/brpop替代前面的lpop/rpop

### springBoot中使用redis集群

```yaml
spring:		
  redis:				
    cluster:					
      nodes:								
      -	139.224.200.249:7000								
      -	139.224.200.249:7001								
      -	139.224.200.249:7002								
      -	139.224.200.249:7003														
      -	139.224.200.249:7005
```

<!-- more -->