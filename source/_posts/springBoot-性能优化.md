---
layout: post
title: springBoot --- 性能优化
date: 2018-06-01 07:16:05
tags: springBoot
categories: springBoot
---

### 服务器启动内存优化
推荐使用 SpringBoot之Undertow 代替Tomcat作为web服务器。可以用JDK bin下面的 `jvisualvm.exe`工具查看。
```xml
<!--web 模块-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
	<!--排除tomcat依赖-->
		<exclusion>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<groupId>org.springframework.boot</groupId>
		</exclusion>
	</exclusions>
</dependency>
<!--undertow容器-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```
<!-- more -->