---
layout: post
title: Spring Boot集成Spring Security
date: 2018-02-20 03:26:57
tags: springBoot
categories: springBoot
---

### Redisson分布式锁
1、添加依赖：
```xml
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.7.5</version>
</dependency>
```
2、定义Lock接口：
```java
import java.util.concurrent.TimeUnit;
import org.redisson.api.RLock;

public interface DistributedLocker {
    RLock lock(String lockKey);
    RLock lock(String lockKey, int timeout);
    RLock lock(String lockKey, TimeUnit unit, int timeout);
    boolean tryLock(String lockKey, TimeUnit unit, int waitTime, int leaseTime);
    void unlock(String lockKey);
    void unlock(RLock lock);
}
```
3、定义Lock接口实现类：
```java
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import java.util.concurrent.TimeUnit;

public class RedissonDistributedLocker implements DistributedLocker {
    
    private RedissonClient redissonClient;

    @Override
    public RLock lock(String lockKey) {
        RLock lock = redissonClient.getLock(lockKey);
        lock.lock();
        return lock;
    }

    @Override
    public RLock lock(String lockKey, int leaseTime) {
        RLock lock = redissonClient.getLock(lockKey);
        lock.lock(leaseTime, TimeUnit.SECONDS);
        return lock;
    }
    
    @Override
    public RLock lock(String lockKey, TimeUnit unit ,int timeout) {
        RLock lock = redissonClient.getLock(lockKey);
        lock.lock(timeout, unit);
        return lock;
    }
    
    @Override
    public boolean tryLock(String lockKey, TimeUnit unit, int waitTime, int leaseTime) {
        RLock lock = redissonClient.getLock(lockKey);
        try {
            return lock.tryLock(waitTime, leaseTime, unit);
        } catch (InterruptedException e) {
            return false;
        }
    }
    
    @Override
    public void unlock(String lockKey) {
        RLock lock = redissonClient.getLock(lockKey);
        lock.unlock();
    }
    
    @Override
    public void unlock(RLock lock) {
        lock.unlock();
    }

    public void setRedissonClient(RedissonClient redissonClient) {
        this.redissonClient = redissonClient;
    }
}
```
4、定义自动装配类：
```java
@Configuration
public class RedissonConfig{

    @Value("${spring.redis.host}")
    private String host;
    @Value("${spring.redis.port}")
    private int port;
    @Value("${spring.redis.timeout}")
    private int timeout;
    @Value("${spring.redis.password}")
    private String password;

    RedissonConfig(){}

//    /**
//     * 哨兵模式自动装配
//     * @return
//     */
//    @Bean
//    @ConditionalOnProperty(name="redisson.master-name")
//    RedissonClient redissonSentinel() {
//        Config config = new Config();
//        SentinelServersConfig serverConfig = config.useSentinelServers()
//                .addSentinelAddress("10.47.91.83:26379,10.47.91.83:26380,10.47.91.83:26381")
//                .setMasterName("mymaster")
//                .setTimeout(6400)
//                .setMasterConnectionPoolSize(10)
//                .setSlaveConnectionPoolSize(8);
//
//        if(StringUtils.isNotBlank(password)) {
//            serverConfig.setPassword(password);
//        }
//        return Redisson.create(config);
//    }

    /**
     * 单机模式自动装配
     * @return
     */
    @Bean
    RedissonClient redissonSingle() {
        Config config = new Config();
        config.useSingleServer()
                .setAddress("redis://" + host + ":" + port)
                .setTimeout(timeout)
                .setConnectionPoolSize(64)
//                .setPassword(password)
                .setConnectionMinimumIdleSize(10);
        return Redisson.create(config);
    }

    /**
     * 装配locker类，并将实例注入到RedissLockUtil中
     * @return
     */
    @Bean
    DistributedLocker distributedLocker(RedissonClient redissonClient) {
        DistributedLocker locker = new RedissonDistributedLocker();
        locker.setRedissonClient(redissonClient);
        RedissLockUtil.setLocker(locker);
        return locker;
    }

}
```
5、Lock工具类：
```java
import java.util.concurrent.TimeUnit;
import org.redisson.api.RLock;
import DistributedLocker;

/**
 * redis分布式锁帮助类
 * @author yangzhilong
 *
 */
public class RedissLockUtil {
    private static DistributedLocker redissLock;
    
    public static void setLocker(DistributedLocker locker) {
        redissLock = locker;
    }
    
    /**
     * 加锁
     * @param lockKey
     * @return
     */
    public static RLock lock(String lockKey) {
        return redissLock.lock(lockKey);
    }

    /**
     * 释放锁
     * @param lockKey
     */
    public static void unlock(String lockKey) {
        redissLock.unlock(lockKey);
    }
    
    /**
     * 释放锁
     * @param lock
     */
    public static void unlock(RLock lock) {
        redissLock.unlock(lock);
    }

    /**
     * 带超时的锁
     * @param lockKey
     * @param timeout 超时时间   单位：秒
     */
    public static RLock lock(String lockKey, int timeout) {
        return redissLock.lock(lockKey, timeout);
    }
    
    /**
     * 带超时的锁
     * @param lockKey
     * @param unit 时间单位
     * @param timeout 超时时间
     */
    public static RLock lock(String lockKey, TimeUnit unit ,int timeout) {
        return redissLock.lock(lockKey, unit, timeout);
    }
    
    /**
     * 尝试获取锁
     * @param lockKey
     * @param waitTime 最多等待时间
     * @param leaseTime 上锁后自动释放锁时间
     * @return
     */
    public static boolean tryLock(String lockKey, int waitTime, int leaseTime) {
        return redissLock.tryLock(lockKey, TimeUnit.SECONDS, waitTime, leaseTime);
    }
    
    /**
     * 尝试获取锁
     * @param lockKey
     * @param unit 时间单位
     * @param waitTime 最多等待时间
     * @param leaseTime 上锁后自动释放锁时间
     * @return
     */
    public static boolean tryLock(String lockKey, TimeUnit unit, int waitTime, int leaseTime) {
        return redissLock.tryLock(lockKey, unit, waitTime, leaseTime);
    }
}
```