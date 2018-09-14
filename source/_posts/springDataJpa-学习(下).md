---
layout: post
title: springDataJpa学习(下)
date: 2018-02-09 04:06:09
tags: springDataJpa
categories: springDataJpa
---

## 多数据源的支持

### 同源数据库的多源支持

主要分为三步：
* 1.配置多数据源
* 2.不同源的实体类放入不同包路径
* 3.声明不同的包路径下使用不同的数据源、事务支持
参考：[Spring Boot多数据源配置与使用](https://www.jianshu.com/p/34730e595a8c)

### 异构数据库多源支持
实体类声明`@Entity`关系型数据库支持类型,
声明`@Document`为mongodb支持类型，不同的数据源使用不同的实体即可

```java
//方式1
@Entity
public class Person {
  …
}
interface PersonRepository extends Repository<Person, Long> {
 …
}

//方式2
@Document
public class User {
  …
}
interface UserRepository extends Repository<User, Long> {
 …
}
```

如果User用户既使用mysql也使用mongodb呢，也可以做混合使用

<!-- more -->

```java
@Entity
@Document
public class Person {
  …
}
interface JpaPersonRepository extends Repository<Person, Long> {
 …
}

interface MongoDBPersonRepository extends Repository<Person, Long> {
 …
}
```

还有一种简单的配置，限制包的路径，比如A包路径下使用mysql,B包路径下使用mongoDB

```java
@EnableJpaRepositories(basePackages = "com.neo.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.neo.repositories.mongo")
interface Configuration { }
```

### 其他

1、使用`@Enumerated(EnumType.STRING)`注解，在数据库中存储的枚举对应的String类型

```java
@Enumerated(EnumType.STRING) 
@Column(nullable = true)
private UserType type;
```

2、使用`@Transient`注解，表示不需要和数据库相映射，该字段不存入数据库

```java
@Transient
private String resCode; //用于辅助计算
```