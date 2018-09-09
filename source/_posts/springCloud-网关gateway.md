---
layout: post
title: springCloud--网关zuul
date: 2018-07-16 01:10:35
tags: springCloud
categories: springCloud
---

### 开始Gateway 限流

1、添加依赖：
```xml
<!--spring cloud gateway依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--基于 reactive stream 的redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

2、配置IP的限流请求
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route        #路由的唯一id，不定义的话为一个uuid
        uri: lb://pigx-upms            #http请求为lb://前缀 + 服务id；ws请求为lb:ws://前缀 + 服务id；表示将请求负载到哪一个服务上
        order: 10000
        predicates:                   #路由的请求匹配规则
        - Path=/admin/**             
        filters:                       #请求转发前的filter
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 1  # 令牌桶的容积
            redis-rate-limiter.burstCapacity: 3  # 流速 每秒
            key-resolver: "#{@remoteAddrKeyResolver}" #SPEL表达式去的对应的bean
        - StripPrefix=1
```

<!-- more -->

`predicates`：请求匹配规则，为一个数组，每个规则为并且的关系。包含两个参数：

 - 1. `name`：规则名称，目前有10个，有Path，Query，Method，Header，After，Before，Between，Cookie，Host，RemoteAddr  
 - 2. `args`：参数key-value键值对，可简写，如上面形式。

`filters`：请求过滤filter，为一个数组，每个filter都会顺序执行。包含两个参数： 
- 1. `name`：过滤filter名称，常用的有Hystrix断路由，RequestRateLimiter限流，StripPrefix截取请求url 
- 2. `args`：参数key-value键值对

3、配置Bean，多维度限流入口

```java
/**
* 自定义限流标志的key，多个维度可以从这里入手
* exchange对象中获取服务ID、请求信息，用户信息等
*/
@Bean
KeyResolver remoteAddrKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
```

4、添加Hystrix断路由（可选）：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      routes:
      - id: i5xforyou-biz-auth
        uri: lb://i5xforyou-biz-auth
        predicates:
        - Path= ${server.servlet.context-path}/auth/**
        filters:
        - StripPrefix= 1
        - name: Hystrix
          args:
            name: authHystrixCommand               #触发断路由后的跳转请求
            fallbackUri: forward:/hystrixTimeout     #触发断路由后的跳转请求url

#设置断路由的超时时间，毫秒
hystrix：
  command：
	default：
	  execution：
		isolation：
		  thread：
		    timeoutInMilliseconds：30000
```
```java
## authHystrixCommand类
@RestController
public class HystrixCommandController {
    protected final Logger log = LoggerFactory.getLogger(this.getClass());

    @RequestMapping("/hystrixTimeout")
    public JsonPackage hystrixTimeout() {
        log.error("i5xforyou-service-gateway触发了断路由");
        return JsonPackage.getHystrixJsonPackage();
    }

    @HystrixCommand(commandKey="authHystrixCommand")
    public JsonPackage authHystrixCommand() {
        return JsonPackage.getHystrixJsonPackage();
    }

}
```





