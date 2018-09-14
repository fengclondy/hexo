---
layout: post
title: springBoot-系统状态监控
date: 2018-08-07 08:23:35
tags: springBoot
categories: springBoot
---

### 使用

1、引入依赖：
```xml
<dependency>    
  <groupId>org.springframework.boot</groupId>    
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

如果加入了security依赖，则所有的接口默认都需要被验证，如果只想 /admin路径下的请求进行验证，则需要加入配置
```properties
security.basic.enabled=true
security.basic.path=/admin
security.user.name=admin
security.user.password=password
```

actuator暴露的health接口权限是由两个配置：`management.security.enabled`和 `endpoints.health.sensitive`组合的结果进行返回的。

<!-- more -->

```yaml
# actuator不需要安全保证
management:
  #port: 54001  // 指定监听端口，不指定则语server端口一致
  security:
    enabled: false
#actuator的health接口不需要安全保证
endpoints:
  health:
    sensitive: false
```

2、info配置（可选）
```yaml
info:
  app:
    name: "@project.name@" #从pom.xml中获取
    description: "@project.description@"
    version: "@project.version@"
    spring-boot-version: "@project.parent.version@"
```
