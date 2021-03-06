---
layout: post
title: springBoot经典小例子
date: 2018-01-26 07:46:04
tags: springBoot
categories: springBoot
---

### 新版本特性

#### JDK 8

直接从JDK 8起跳。身为写过C#的程序员，语言内置的函数式编程支持实在是太舒服了。如果你的JDK还在6或者7，就享受不到这种便利。不过，不清楚国内到底有多少生产系统的JDK还停留在6或者7，反正我司已经全部从8起跳了。

#### HikariCP

默认连接池改为HikariCP。Tomcat JDBC Connection Pool是一个“有年头”的连接池了，业界使用也很广泛。但是“江山代有人才出”，根据测试数据，新出现的HikariCP相比Tomcat性能有显著的提升，有些指标甚至是数量级的提升。而且，HikariCP的性能并不因为并发而变化，这也是很有吸引力的。

#### Metrics

如今Metrics已经是系统的“标配”了，Spring Boot 2引入了Micrometer，来统一metrics的规范，使得开发人员更好的理解和使用metrics的模块。RabbitMQ、JVM 线程和垃圾收集指标会自动进行 instrument 监控，异步控制器（controller）也会自动添加到监控里。

<!-- more -->

#### Lettuce

Lettuce取代了Jedis，成为底层连接Redis的方案。Lettuce是一个可伸缩的线程安全的 Redis 客户端，用于同步、异步和响应式使用。多个线程可以共享同一个 RedisConnection，避免了Jedis的线程不安全问题。

>Jedis在实现上是直连redis server，多线程环境下非线程安全，除非使用连接池，为每个Jedis实例增加物理连接。Lettuce基于Netty的连接实例（StatefulRedisConnection），可以在多个线程间并发访问，且线程安全，满足多线程环境下的并发访问，同时它是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例**。

```properties
spring.redis.host=localhost
spring.redis.port=6379
#spring.redis.password=root #根据需要
# 连接超时时间（毫秒）
spring.redis.timeout=10000
# Redis默认情况下有16个分片，这里配置具体使用的分片，默认是0
spring.redis.database=0
# 连接池最大连接数（使用负值表示没有限制） 默认 8
spring.redis.lettuce.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
spring.redis.lettuce.pool.max-wait=-1
# 连接池中的最大空闲连接 默认 8
spring.redis.lettuce.pool.max-idle=8
# 连接池中的最小空闲连接 默认 0
spring.redis.lettuce.pool.min-idle=0
```

```java
@Configuration
@AutoConfigureAfter(RedisAutoConfiguration.class)
public class RedisCacheAutoConfiguration {

    @Bean
    public RedisTemplate<String, Serializable> redisCacheTemplate(LettuceConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Serializable> template = new RedisTemplate<>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```

```java
public class User implements Serializable {
    private static final long serialVersionUID = 8655851615465363473L;
    private Long id;
    private String username;
    private String password;
    // TODO  省略get set
}
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Chapter8ApplicationTest {

    @Autowired
    private RedisTemplate<String, Serializable> redisCacheTemplate;

	//@Autowired
   // private StringRedisTemplate<String, String> stringRedisTemplate;

    @Test
    public void get() {
    
         // 2.0之前的redisTemplate线程安全的实现,需要构造线程池
        //ExecutorService executorService = Executors.newFixedThreadPool(1000);
        //IntStream.range(0, 1000).forEach(i ->
                //executorService.execute(() -> stringRedisTemplate.opsForValue().increment("kk", 1));
        //);
        //stringRedisTemplate.opsForValue().set("k1", "v1");
        //final String k1 = stringRedisTemplate.opsForValue().get("k1");
        //log.info("[字符缓存结果] - [{}]", k1);
    
        String key = "battcn:user:1";
        redisCacheTemplate.opsForValue().set(key, new User(1L, "u1", "pa"));
        final User user = (User) redisCacheTemplate.opsForValue().get(key);
        log.info("[对象缓存结果] - [{}]", user);
    }
}
```


#### 支持HTTP/2

如果你留意访问信息，会发现不少大厂已经悄然升级到HTTP/2了，其中升级最多的是资源类站点，毕竟二进制传输和首部压缩之类的特性可以直接享受。但是要想享受HTTP/2的全部好处，程序上不直接照搬原来的代码是不可能的，起码Server Push之类就做不到。Spring Boot 2.0中，无论选择Tomcat, Netty, Jetty哪一个作为Web容器，都可以支持HTTP/2。但是注意，如果要使用HTTP/2的全部特性，必须使用Servlet 4.0及以上版本。JDK 9新增的HTTPClient可以完整支持HTTP/2。

当然，Spring Boot 2.0的新特性还不止这些，还有OAuth 2.0、嵌入式Netty、Kotlin、JOOQ、Quartz、WebFlux响应式编程等等。
