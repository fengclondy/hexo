---
layout: post
title: redis入门(1)
date: 2018-03-15 07:56:47
tags: redis
categories: redis
---

### 简介

Redis是目前业界使用最广泛的内存数据存储。
相比memcached，Redis支持更丰富的数据结构，例如hashes, lists, sets等，同时支持数据持久化。
除此之外，Redis还提供一些类数据库的特性，比如事务，HA，主从库。
可以说Redis兼具了缓存系统和数据库的一些特性，因此有着丰富的应用场景。
本文介绍Redis在Spring Boot中两个典型的应用场景。

在互联网场景下，尤其 2C 端大流量场景下，需要将一些经常展现和不会频繁变更的数据，存放在存取速率更快的地方。
缓存就是一个存储器，在技术选型中，常用 Redis 作为缓存数据库。
缓存主要是在获取资源方便性能优化的关键方面。

### 流程

大致流程如下：

获取商品详情举例

a. 从商品 Cache 中获取商品详情，如果存在，则返回获取 Cache 数据返回。

b. 如果不存在，则从商品 DB 中获取。获取成功后，将数据存到 Cache 中。则下次获取商品详情，就可以从 Cache 就可以得到商品详情数据。

c. 从商品 DB 中更新或者删除商品详情成功后，则从缓存中删除对应商品的详情缓存
![](http://p2jr3pegk.bkt.clouddn.com/redis01-1.png)

<!-- more -->

### 使用场景说明

#### 计数器

数据统计的需求非常普遍，通过原子递增保持计数。例如，点赞数、收藏数、分享数等。

#### 排行榜

排行榜按照得分进行排序，例如，展示最近、最热、点击率最高、活跃度最高等等条件的top list。

#### 用于存储时间戳

类似排行榜，使用redis的zset用于存储时间戳，时间会不断变化。例如，按照用户关注用户的最新动态列表。

#### 记录用户判定信息

记录用户判定信息的需求也非常普遍，可以知道一个用户是否进行了某个操作。例如，用户是否点赞、用户是否收藏、用户是否分享等。

#### 社交列表

社交属性相关的列表信息，例如，用户点赞列表、用户收藏列表、用户关注列表等。

#### 缓存

缓存一些热点数据，例如，PC版本文件更新内容、资讯标签和分类信息、生日祝福寿星列表。

#### 队列

Redis能作为一个很好的消息队列来使用，通过list的lpop及lpush接口进行队列的写入和消费，本身性能较好能解决大部分问题。但是，不提倡使用，更加建议使用rabbitmq等服务，作为消息中间件。

#### 会话缓存

使用Redis进行会话缓存。例如，将web session存放在Redis中。


### 安装与使用

1、下载地址：https://github.com/MSOpenTech/redis/releases

2、开一个cmd窗口，使用cd命令切换至安装目录，运行（默认配置）： `redis-server.exe`

![](http://p2jr3pegk.bkt.clouddn.com/redis01.png)

windows下可能会出现闪退情况，原因是redis服务需要同时启动两个服务。
解决办法是,新建start.bat文件，在里面填入：`redis-server.exe  redis.windows.conf`

如果想要加入系统自启动，则需编写xxx.cmd文件：
```
@echo off
echo Starting rabbitMQ...
start "rabbitMQ" "C:\Program Files\RabbitMQ Server\rabbitmq_server-3.7.4\sbin\rabbitmq-server.bat"
echo Starting redis...
start "redis" "startRedis.bat"
```

或者：`redis-server --service-install redis.windows-service.conf --loglevel verbose`。你也可以在`redis.windows.conf`中设置密码。列举常用服务：
```
redis-server --service-uninstall    #卸载服务
redis-server --service-start    #开启服务
redis-server --service-stop    #停止服务
```



3、在打开一个cmd窗口，进行基本操作：

```
redis-cli.exe -h 127.0.0.1 -p 6379 

设置键值对 set myKey abc

取出键值对 get myKey
```

4、其他的一些相关操作：

```
keys *          匹配查找所有的key
flushall        清空所有的key
```

配置文件详解：https://www.cnblogs.com/DreamDrive/p/5587219.html


5、配置密码,在.config文件里配置：
```
requirepass [密码]；
```

此时，连接时，出现没有权限提示，输入密码即可：
```
>auth [密码]
```

### 性能测试 benchmark 
命令行输入：`redis-benchmark -h 127.0.0.1 -p 6379 -c 1000 -n 100000`
表示向server发送10万个请求，每次请求并发数为1000。结果如下：
```
 ====== PING_INLINE ======
    100000 requests completed in 154.46 seconds
    1000 parallel clients
    3 bytes payload
    keep alive: 1

    647.43 requests per second

====== PING_INLINE ======
```
表示平均每秒处理647.43个并发请求，相当于每秒有60万用户请求。此电脑配置4G内存单核CUP，性能还是能够的。





### Redis 数据结构简介

1. Redis 可以存储键与5种不同数据结构类型之间的映射，

这5种数据结构类型分别为String（字符串）、List（列表）、Set（集合）、Hash（散列）和 Zset（有序集合）。

2. spring 封装了 RedisTemplate 对象来进行对redis的各种操作，它支持所有的 redis 原生的 api。  

RedisTemplate中定义了对5种数据结构操作：

```
redisTemplate.opsForValue();   //操作字符串
redisTemplate.opsForHash();   //操作hash
redisTemplate.opsForList();   //操作list
redisTemplate.opsForSet();   //操作set
redisTemplate.opsForZSet();  //操作有序set
```

### 使用

>使用springBoot 1.5创建

1、加入依赖：

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-redis</artifactId>  
</dependency> 
```

2、添加配置文件：

```
properties中加入配置
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0  
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379  
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8  
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1  
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8  
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0  
# 连接超时时间（毫秒）
spring.redis.timeout=0
```

3、在你的类中注入：

```
   @Autowired
   private StringRedisTemplate stringRedisTemplate;
```

或者：

```
    @Autowired
    private RedisTemplate redisTemplate;
```

4、使用

```
//方式1：key-value形式
使用：redisTemplate.opsForValue().set("name","tom");
结果：redisTemplate.opsForValue().get("name")  输出结果为tom

//方式2：list集合形式
存：redisTemplate.opsForSet().add("onlineUsers",userName);
取：redisTemplate.opsForSet().members("onlineUsers")

//删除
redisTemplate.delete("onlineUsers");
```




编写逻辑代码（一个简单的方法事例）

编写controller：

```
    @Autowired
    UserServiceIMPL userServiceImpl;
    
    @PostMapping("/findUserById/{id}")
    public String findUserNameById(@PathVariable("id") Long id) {
        String userName=userServiceImpl.findUserNameById(id);
        return userName;
    }
```

编写service：

```
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 获取用户逻辑：
     * 如果缓存存在，从缓存中获取用户的名字（这里注意只存了一个value值）
     * 如果缓存不存在，从 DB 中获取用户名字，然后插入缓存
     */
    public String findUserNameById(Long id) {
        // 从缓存中获取用户信息
        String key = "user_" + id;
        ValueOperations operations = redisTemplate.opsForValue();
        //ValueOperations operations1 = redisTemplate.op

        // 缓存存在
        boolean hasKey = redisTemplate.hasKey(key);
        if (hasKey) {
            Object value = operations.get(key);
            System.out.print(" 从缓存中获取了用户 >> " + value.toString());
            return value.toString();
        }else{
            // 从 DB 中获取城市信息
            User user = userRepository.findOne(id);
            // 插入缓存
            operations.set(key, user.getUserName(), 10, TimeUnit.MINUTES ); //设置10分钟失效时间
            System.out.print("用户插入缓存 >> " + user.getUserName());
            return user.getUserName();
        }
    }
```

编写repository：

```
public interface UserRepository extends JpaRepository<User, Long> {
}
```

>注意项目正常启动的前提是安装了redis服务，并开启


### 注意事项

1、java代码：添加一个集合元素：`redisTemplate.opsForSet().add("key","zhangsan")`

等价于在控制台输入如下语句

```
>sadd ddd "\"zhangsan\""
```
>这里需要特别注意的是一定要加双引号将要插入的值括起来，且双引号需要用转义字符\。






https://www.jianshu.com/p/7bf5dc61ca06















-------------------------------------------------------下面删除------------------------

3、编写Cache配置类：

```java
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport{
    
    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

    @SuppressWarnings("rawtypes")
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
        //设置缓存过期时间
        //rcm.setDefaultExpiration(60);//秒
        return rcm;
    }
    
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

}
```

4、编写测试类：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(Application.class)
public class TestRedis {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    
    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void test() throws Exception {
        stringRedisTemplate.opsForValue().set("aaa", "111");
        Assert.assertEquals("111", stringRedisTemplate.opsForValue().get("aaa"));
    }
    
    @Test
    public void testObj() throws Exception {
        User user=new User("aa@126.com", "aa", "aa123456", "aa","123");
        ValueOperations<String, User> operations=redisTemplate.opsForValue();
        operations.set("com.neox", user);
        operations.set("com.neo.f", user,1,TimeUnit.SECONDS);
        Thread.sleep(1000);
        //redisTemplate.delete("com.neo.f");
        boolean exists=redisTemplate.hasKey("com.neo.f");
        if(exists){
            System.out.println("exists is true");
        }else{
            System.out.println("exists is false");
        }
       // Assert.assertEquals("aa", operations.get("com.neo.f").getUserName());
    }
}
```