---
layout: post
title: springBoot-数据库连接池后起之秀HikariCP
date: 2018-08-07 08:55:47
tags: springBoot
categories: springBoot
---

### 数据库连接池
springBoot默认集成了tomcat，所以默认也就有了tomcat连接池。为了优化连接池效率和方法，采用HikariCP
```
<!--tomcat-jdbc排除 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.apache.tomcat</groupId>
			<artifactId>tomcat-jdbc</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<!--HikariCP连接池-->
<dependency>
	<groupId>com.zaxxer</groupId>
	<artifactId>HikariCP</artifactId>
</dependency>
```
