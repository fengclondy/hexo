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
@Cacheable	执行方法前先查询缓存是否有数据。有则直接返回缓存数据；否则查询数据再将数据放入缓存
@CachePut	执行新增或更新方法后，将数据放入缓存中
@CacheEvict	清除缓存
@Caching	将多个缓存操作重新组合到一个方法中
```

https://juejin.im/post/59408d5aa0bb9f006b725f50