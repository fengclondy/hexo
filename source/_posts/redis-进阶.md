---
layout: post
title: redis进阶
date: 2018-03-16 03:03:00
tags: redis
categories: redis
---

### 使用

1、添加依赖和配置（此处省略）

2、开启缓存支持

```JAVA
@Configuration
@EnableCaching
public class CacheConfiguration {}
```

3、创建服务类

```JAVA
@Service("concurrenmapcache.cacheService")
public class CacheService {

}
```

@Cacheable 注解。@Cacheable 在方法执行前 Spring 先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，调用方法并将方法返回值放进缓存。有两个重要的值， value，返回的内容将存储在 value 定义的缓存的名字对象中。key，如果不指定将使用默认的 KeyGenerator 生成。

<!-- more -->

(1)在查询方法上，添加 @Cacheable 注解，其中缓存名称为 concurrenmapcache

```JAVA
@Cacheable(value = "concurrenmapcache")
public long getByCache() {
}
```

(2)@CachePut 与 @Cacheable 类似，都会将方法的返回值放到缓存中, 主要用于数据新增和修改方法。

```JAVA
@CachePut(value = "concurrenmapcache")
public long save() {
}
```

(3)@CacheEvict 将一条或多条数据从缓存中删除, 主要用于删除方法，用来从缓存中移除相应数据。

```JAVA
@CacheEvict(value = "concurrenmapcache")
public void delete() {
}
```

### 其他注解

```
@EnableCaching	开启缓存功能，放在配置类或启动类上
@CacheConfig	缓存配置，设置缓存名称
@Cacheable	执行方法前先查询缓存是否有数据。有则直接返回缓存数据；否则查询数据再将数据放入缓存，有三个属性，value、key和condition，condition默认为空
@CachePut	执行新增或更新方法后，将数据放入缓存中，与@Cacheable先查缓存在执行方法不同，它是先执行方法
@CacheEvict	清除缓存，可以指定的属性有value、key、condition、allEntries(清除缓存中的所有元素,默认为false)和beforeInvocation(是否在执行之前清除，避免执行报错，默认为false)
@Caching	将多个缓存操作重新组合到一个方法中，有三个属性：cacheable、put和evict，分别用于指定@Cacheable、@CachePut和@CacheEvict
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



https://juejin.im/post/59408d5aa0bb9f006b725f50