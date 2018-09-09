---
layout: post
title: springboot微服务下常见问题
date: 2018-05-16 03:32:17
tags: study
categories: study
---

### 常见问题

### IDEA采用自己的maven配置

`~./conf` 目录下配置配置文件：

```
### 设置自己的maven仓库保存地址
<localRepository>D:\Program Files\apache-maven-3.5.2\george</localRepository>

### 添加阿里云镜像
<mirrors>
  <mirror> 
    <id>alimaven</id> 
    <name>aliyun maven</name> 
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url> 
    <mirrorOf>central</mirrorOf> 
  </mirror> 
</mirrors>

### 设置全局JDK
<profile>  
    <id>jdk18</id>  
    <activation>  
        <activeByDefault>true</activeByDefault>  
        <jdk>1.8</jdk>  
    </activation>  
    <properties>  
        <maven.compiler.source>1.8</maven.compiler.source>  
        <maven.compiler.target>1.8</maven.compiler.target>  
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
    </properties>   
</profile>  
```

#### RESTful API

五种请求类型：

```
GET ：获取资源
POST ：新建资源
PUT ：更新资源（客户端提供完整的资源）
UPDATE ：更新资源（客户端提供改变的属性）
DELETE ：删除资源
```
>RESTful的含义：url地址中只包含名词表示资源，使用http动词表示动作进行操作资源。
例子：
```
### 错误   -->  正确
GET /blog/getArticles --> GET /blog/Articles  获取所有文章
GET /blog/addArticles --> POST /blog/Articles  添加一篇文章
GET /blog/editArticles --> PUT /blog/Articles  修改一篇文章 
GET /rest/api/deleteArticles?id=1 --> DELETE /blog/Articles/1  删除一篇文章
```

#### 请求注解

```
@RequestParam   --针对get请求（无Body的请求）
@RequestBody   --针对post请求
@RequestHeader    --请求的头部校验
```


#### 将linkedHashMap转换成对象Object

>ObjectMapper是Jackson提供的一个类，作用是将java对象与json格式相互转化

list是传入的linkedHashMap格式数据
```
ObjectMapper mapper = new ObjectMapper();
List<User> infos = mapper.convertValue(list, new TypeReference<List<User>>(){});
```

#### 事务管理

在默认的代理模式下，只有目标方法由外部调用，才能被 Spring 的事务拦截器拦截。在同一个类中的两个方法直接调用，是不会被 Spring 的事务拦截器拦截，
就像上面的 save 方法直接调用了同一个类中的 method1方法，method1 方法不会被 Spring 的事务拦截器拦截


#### 工程名的构建
```
spring.application.name=XXX-XXX-XX
```
不能识别下划线"_"，只能识别"-"


#### 为表添加唯一性约束
```
ALTER TABLE msg_queue ADD UNIQUE KEY(resource_name, resource_type);
```

### 微服务下安全认证与鉴权

1、单体应用体系下，应用是一个整体，一般针对所有的请求都会进行权限校验。
请求一般会通过一个权限的拦截器进行权限的校验，在登录时将用户信息缓存到 session 中，后续访问则从缓存中获取用户信息。

2、微服务架构下，一个应用会被拆分成若干个微应用，每个微应用都需要对访问进行鉴权，每个微应用都需要明确当前访问用户以及其权限。
尤其当访问来源不只是浏览器，还包括其他服务的调用时，单体应用架构下的鉴权方式就不是特别合适了。
在微服务架构下，要考虑外部应用接入的场景、用户 - 服务的鉴权、服务 - 服务的鉴权等多种鉴权场景。