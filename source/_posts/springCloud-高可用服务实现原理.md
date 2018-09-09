---
layout: post
title: springBoot--高可用服务实现原理
date: 2018-08-06 01:39:49
tags:
---

### 高可用的设计原则
- AKF拆分原则
- 前后端分离
- 无状态服务
- Restful通信风格

####  AKF拆分原则
拆分方式：按照不同服务功能进行拆分
拆分要点：高内聚，低耦合。一个服务尽量完成一个功能，减少服务间的频繁调用。

#### 前后端分离
前端专注页面展示，布局以及多场景集成。后端提供统一的数据和模型供前端访问。

#### 无状态服务
如果一个数据需要被多个服务共享，才能完成一笔交易，那么这个数据被称为状态。进而依赖这个“状态”数据的服务被称为有状态服务，反之称为无状态服务。简单点说就是，应该把缓存的数据迁移到分布式缓存中存储，让业务服务变成一个无状态的计算节点

#### Restful通信
无状态协议HTTP，扩展能力很强。JSON 报文序列化，轻量简单。语言无关，支持广泛。

>当然，微服务也带来了一系列的问题：分布式事务的处理（主要指的是数据最终达到一直状态），依赖服务不稳定，模块重复构建等。

### 高可用服务注册中心

构建双节点(多结点)的服务注册中心,通过进行互相注册的方式来实现高可用的部署。
```
##application-peer1.properties
spring.application.name=eureka-server
server.port=1111
eureka.instance.hostname=peer1
eureka.client.serviceUrl.defaultZone=http://peer2:1112/eureka/
```
```
## application-peer2.properties
spring.application.name=eureka-server
server.port=1112
eureka.instance.hostname=peer2
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka/
```

在host文件里面添加：
```
127.0.0.1 peer1
127.0.0.1 peer2
```

指定程序启动时的一些额外配置：
```
java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer1
java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer2
```

在微服务的配置文件里指明,即可实现微服务的双向同时注册
```
spring.application.name=compute-service
server.port=2222
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka/,http://peer2:1112/eureka/
```
一般构建集群服务，都是通过两两相互注册，形成环路，实现集群中节点完全对等的效果，即当某一个节点故障不可用时，仍然可以通过其他节点进行访问。
常用的架构图为：![](http://p2jr3pegk.bkt.clouddn.com/springCloud05-1.png)


### 服务熔断、限流、降级处理
主要用于解决高并发时系统的可靠性。

#### 服务熔断
在微服务架构中，当调用某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。
这种方法，可以避免系统崩溃，引起"雪崩效应"。当检测到该节点微服务调用响应正常后，恢复调用链路。

在Spring Cloud框架里，熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。
（在dubbo中也可利用nio超时+失败次数做熔断。 ）

#### 服务降级
从RPC调用环节来讲，就是去访问一个本地的伪装者而不是真实的服务。通常用@Component去实现@Feign的接口，自定义不可用的返回值。
```
@FeignClient(value = "yl-user-admin", fallback = UserInfoFeignServiceImpl.class)
public interface UserInfoFeign {

    @RequestMapping(value = "api/user/info", method = RequestMethod.GET)
    List getUserInfo(@RequestParam(value = "username") String username);
}


@Component
@Slf4j
public class UserInfoFeignServiceImpl implements UserInfoFeign {
   
    public List getUserInfo(String username) {
        logger.warn("userAdmin服务不可用！");
        return null;
    }
}
```

最后，一定不要忘了：
```
###application.yml
feign:
  hystrix:
    enabled: true
  client:
    config:
      default:
        connectTimeout: 6000
        readTimeout: 6000
        loggerLevel: basic
```

#### 服务限流
可以概括为：
- 对请求的目标URL进行限流（例如：某个URL每分钟只允许调用多少次）
- 对客户端的访问IP进行限流（例如：某个IP每分钟只允许请求多少次）
- 对某些特定用户或者用户组进行限流（例如：非VIP用户限制每分钟只允许调用100次某个API等）
- 多维度混合的限流。此时，就需要实现一些限流规则的编排机制。与、或、非等关系。

限制总并发数（比如数据库连接池、线程池）、限制瞬时并发数（如nginx的limit_conn模块，用来限制瞬时并发连接数）、限制时间窗口内的平均速率
（如Guava的RateLimiter、nginx的limit_req模块，限制每秒的平均速率）；其他还有如限制远程接口调用速率、限制MQ的消费速率。
另外还可以根据网络连接数、网络流量、CPU或内存负载等来限流。

常见场景：拒绝服务（定向到错误页或告知资源没有了）、排队或等待（比如秒杀、评论、下单）、降级（返回兜底数据或默认数据，如商品详情页库存默认有货）。

