---
layout: post
title: redis进阶(2)
date: 2018-07-30 01:20:57
tags: redis
categories: redis
---

### Redis分布式锁
使用命令setnx占用锁，并设置一个锁超时时间，防止出现异常时能及时释放锁。
```shell
> setnx lock:codehole true
OK
> expire lock:codehole 5
... do something critical ...
> del lock:codehole
(integer) 1
```
但是这种方式万一在expire执行前出现了其他故障，如断电等就会造成死锁。
所以，在新版本中这样使用：（**推荐方式**）
```shell
> set lock:codehole true ex 5 nx
OK
... do something critical ...
> del lock:codehole
```

<!-- more -->



### 代码中使用redis分布式锁
>以1.5.x版本采用 jedis 连接 redis 为例，需要自己实现锁。获取锁后，其他线程无法获取锁，获取失败会尝试重新回去锁，直到超过设定的超时时间。当然，某个线程获取锁后，如果发生异常，没有释放锁，那么需要设定一个超时时间，超时后需要主动断开。

1、创建锁类：
```java
/**
 * 全局锁，包括锁的名称
 * Created by fangzhipeng on 2017/4/1.
 */
public class Lock {
    private String name;
    private String value;
    public Lock(String name, String value) {
        this.name = name;
        this.value = value;
    }
    public String getName() {
        return name;
    }
    public String getValue() {
        return value;
    }
}
```

2、分布式锁的具体实现：
```java
@Component
public class DistributedLockHandler {

    private static final Logger logger = LoggerFactory.getLogger(DistributedLockHandler.class);
    private final static long LOCK_EXPIRE = 30 * 1000L;//单个业务持有锁的时间30s，防止死锁
    private final static long LOCK_TRY_INTERVAL = 30L;//默认30ms尝试一次
    private final static long LOCK_TRY_TIMEOUT = 20 * 1000L;//默认尝试20s

    @Autowired
    private StringRedisTemplate template;

    /**
     * 尝试获取全局锁
     *
     * @param lock 锁的名称
     * @return true 获取成功，false获取失败
     */
    public boolean tryLock(Lock lock) {
        return getLock(lock, LOCK_TRY_TIMEOUT, LOCK_TRY_INTERVAL, LOCK_EXPIRE);
    }

    /**
     * 尝试获取全局锁
     *
     * @param lock    锁的名称
     * @param timeout 获取超时时间 单位ms
     * @return true 获取成功，false获取失败
     */
    public boolean tryLock(Lock lock, long timeout) {
        return getLock(lock, timeout, LOCK_TRY_INTERVAL, LOCK_EXPIRE);
    }

    /**
     * 尝试获取全局锁
     *
     * @param lock        锁的名称
     * @param timeout     获取锁的超时时间
     * @param tryInterval 多少毫秒尝试获取一次
     * @return true 获取成功，false获取失败
     */
    public boolean tryLock(Lock lock, long timeout, long tryInterval) {
        return getLock(lock, timeout, tryInterval, LOCK_EXPIRE);
    }

    /**
     * 尝试获取全局锁
     *
     * @param lock           锁的名称
     * @param timeout        获取锁的超时时间
     * @param tryInterval    多少毫秒尝试获取一次
     * @param lockExpireTime 锁的过期
     * @return true 获取成功，false获取失败
     */
    public boolean tryLock(Lock lock, long timeout, long tryInterval, long lockExpireTime) {
        return getLock(lock, timeout, tryInterval, lockExpireTime);
    }

    /**
     * 操作redis获取全局锁
     *
     * @param lock           锁的名称
     * @param timeout        获取的超时时间
     * @param tryInterval    多少ms尝试一次
     * @param lockExpireTime 获取成功后锁的过期时间
     * @return true 获取成功，false获取失败
     */
    public boolean getLock(Lock lock, long timeout, long tryInterval, long lockExpireTime) {
        try {
            if (StringUtils.isEmpty(lock.getName()) || StringUtils.isEmpty(lock.getValue())) {
                return false;
            }
            long startTime = System.currentTimeMillis();
            do{
                if (!template.hasKey(lock.getName())) {
                    ValueOperations<String, String> ops = template.opsForValue();
                    ops.set(lock.getName(), lock.getValue(), lockExpireTime, TimeUnit.MILLISECONDS);
                    return true;
                } else {//存在锁
                    logger.debug("lock is exist!！！");
                }
                if (System.currentTimeMillis() - startTime > timeout) {//尝试超过了设定值之后直接跳出循环
                    return false;
                }
                Thread.sleep(tryInterval);
            }
            while (template.hasKey(lock.getName())) ;
        } catch (InterruptedException e) {
            logger.error(e.getMessage());
            return false;
        }
        return false;
    }

    /**
     * 释放锁
     */
    public void releaseLock(Lock lock) {
        if (!StringUtils.isEmpty(lock.getName())) {
            template.delete(lock.getName());
        }
    }
}
```

3、使用：
```java
@Autowired
DistributedLockHandler distributedLockHandler;
Lock lock=new Lock("lockk","sssssssss);
if(distributedLockHandler.tryLock(lock){
	doSomething();
	distributedLockHandler.releaseLock();
}
```

### 延迟队列消息

异步队列使用list集合，命令 lpush 和 rpop 结合使用

但是无法保证ack机制，如果要保证可靠传输，redis做不到，可以使用rabbitmq或者Kafka

延迟队列，即阻塞读与写，用blpop/brpop替代前面的lpop/rpop

### springBoot中使用redis集群

首先要做的是配置Redis集群，参考：[博客](https://www.cnblogs.com/PatrickLiu/p/8458788.html)

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