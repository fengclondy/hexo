---
layout: post
title: springBoot-整合MybatisPlus
date: 2018-08-08 08:46:27
tags: springBoot
categories: springBoot
---

### springboot整和mybatis-plus

1、添加依赖：
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>com.baomidou</groupId>
	    <artifactId>mybatisplus-spring-boot-starter</artifactId>
	<version>${mybatisplus-spring-boot-starter.version}</version>
</dependency>
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus</artifactId>
	<version>${mybatisplus.version}</version>
</dependency>
```
<!-- more -->

2、配置文件：

```yaml
# datasoure默认使用JDBC
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root
    url: jdbc:mysql://127.0.0.1/cloud?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true

#mybaits-plus配置，修改主键类型，mapper.xml、type 别名等
mybatis-plus:
  mapper-locations: classpath:/mapper/*Mapper.xml
  typeAliasesPackage: com.example.sbmp.entity
  global-config:
    #主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
    id-type: 0
    #字段策略 0:"忽略判断",1:"非 NULL 判断"),2:"非空判断"
    field-strategy: 0
    #驼峰下划线转换
    db-column-underline: true
    #刷新mapper 调试神器
    refresh-mapper: true
    #数据库大写下划线转换
    #capital-mode: true
  configuration:
    map-underscore-to-camel-case: true
    cache-enabled: true
```

3、Mapper扫描配置：
```java
@Configuration
@MapperScan("com.example.sbmp.mapper*")
public class MybatisPlusConfig {
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```