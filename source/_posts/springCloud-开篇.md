---
layout: post
title: springCloud--开篇
date: 2018-07-16 01:54:02
tags: springCloud
categories: springCloud
---


### 一定要注意的地方
拿IDEA中的父子项目说明，需要在父pom文件中申明：
```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>${spring-boot.version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
		<dependency>
			<groupId>io.spring.platform</groupId>
			<artifactId>platform-bom</artifactId>
			<version>${spring-platform.version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>${spring-cloud.version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

另外，同样重要的是依赖包的问题，一定要分清是springBoot还是springCLoud。很常见的spring-boot-starter-amqp和spring-cloud-starter-bus-amqp两个包。