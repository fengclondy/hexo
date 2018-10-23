---
layout: post
title: redis缓存
date: 2018-07-20 03:03:00
tags: redis
categories: redis
---

### 使用

>使用redis作为spring的缓存路径，支持spring cache的环境

1、添加依赖和配置

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2、配置redis文件,此处省略

<!-- more -->

3、开启缓存支持

```JAVA
@Configuration
@EnableCaching
@CacheConfig
public class RedisConfig extends CachingConfigurerSupport {

    @Value("${spring.redis.host}")
    private String host;
    @Value("${spring.redis.port}")
    private int port;
    @Value("${spring.redis.timeout}")
    private int timeout;

    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuffer sb = new StringBuffer();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        /** 设置缓存过期时间*/
        /** cacheManager.setDefaultExpiration(10000); */
        return cacheManager;
    }

    @Bean
    public RedisConnectionFactory factory(){
        //如果什么参数都不设置，默认连接本地6379端口
        JedisConnectionFactory factory = new JedisConnectionFactory();
        factory.setPort(port);
        factory.setHostName(host);
        return factory;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        /** 使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）*/
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        /** 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public */
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        /** 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常*/
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        /** 值采用json序列化,StringRedisSerializer来序列化和反序列化redis的key值 */
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setKeySerializer(new StringRedisSerializer());
        /** 设置hash key 和value序列化模式 */
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        /** 开启事务*/
        template.setEnableTransactionSupport(true);
        return template;
    }
```

4、创建服务类

`@Cacheable` 注解。`@Cacheable` 在方法执行前 Spring 先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，调用方法并将方法返回值放进缓存。有两个重要的值 key 和 value，**返回的内容**将存储在 value 定义的缓存的名字对象中。key，如果不指定将使用默认的 KeyGenerator 生成。

<!-- more -->

(1)在查询方法上，添加 @Cacheable 注解，其中缓存名称为 value:key,即“usercache:name”，值为返回的User实体。

```JAVA
// @Cacheable缓存key为name的数据到缓存usercache中
@Cacheable(value = "usercache", key = "#p0")
public User findUser(String name) {
}
```

(2)@CachePut 与 @Cacheable 类似，都会将方法的返回值放到缓存中, 主要用于数据新增和修改方法。

```JAVA
// @CachePut缓存新增的数据到缓存usercache中
@CachePut(value = "usercache", key = "#p0")
public User save(String name, int id) {
}
```

(3)@CacheEvict 将一条或多条数据从缓存中删除, 主要用于删除方法，用来从缓存中移除相应数据。

```JAVA
// @CacheEvict从缓存usercache中删除key为name的数据
@CacheEvict(value = "usercache", key = "#p0")
public void removeUser(String name) {
     System.out.println("删除数据" + name + "，同时清除对应的缓存");
}
```

### 其他注解

```shell
@EnableCaching	  开启缓存功能，放在配置类或启动类上
@CacheConfig	 缓存配置，设置缓存名称
@Cacheable	   执行方法前先查询缓存是否有数据。有则直接返回缓存数据；否则查询数据再将数据放入缓存，有三个属性，value、key和condition，condition默认为空。value是必须的，key支持两种写法，"#参数名"或者"#p参数index"
@CachePut	  执行新增或更新方法后，将数据放入缓存中，与@Cacheable先查缓存在执行方法不同，它是先执行方法
@CacheEvict	    清除缓存，可以指定的属性有value、key、condition、allEntries(清除缓存中的所有元素,默认为false)和beforeInvocation(是否在执行之前清除，避免执行报错，默认为false)
@Caching	  将多个缓存操作重新组合到一个方法中，有三个属性：cacheable、put和evict，分别用于指定@Cacheable、@CachePut和@CacheEvict
```

例子：
```java
### 例子1
@Cacheable(value="users", key="#id")
//@Cacheable(value={"users"}, key="#user.id", condition="#user.id%2==0")
public User find(Integer id) {
   return null;
}
   
### 例子2
@Caching(cacheable = @Cacheable("users"), evict = { @CacheEvict("cache2"),@CacheEvict(value = "cache3", allEntries = true) })
public User find(Integer id) {
   return null;
}
```

>关于key的说明：
| 属性名称        | 描述           | 示例  |
| ------------- |:-------------:| -----:|
| methodName | 当前方法名 | #root.methodName |
| method | 当前被调用的对象 | #root.method.name |
| target | 当前被调用的对象  | #root.target |
| targetClass | 当前被调用的对象的class | #root.targetClass |
| args | 当前方法参数组成的数组 | #root.args[0] |
| caches | 当前被调用的方法使用的Cache | #root.caches[0].name |

>>默认的key生成策略是通过 KeyGenerator (config里面配置的)生成的，其默认策略如下：
>>- 如果方法没有参数，则使用0作为key。
>>- 如果只有一个参数的话则使用该参数作为key。
>>- 如果参数多余一个的话则使用所有参数的hashCode作为key。





https://juejin.im/post/59408d5aa0bb9f006b725f50