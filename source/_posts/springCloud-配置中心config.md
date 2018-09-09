---
layout: post
title: springCloud-配置中心
date: 2018-08-07 08:04:40
tags: springCloud
categories: springCloud
---

### SpringCloud配置中心

开发分布式系统如果还是各个服务配置文件单独配置肯定是不行的，springcloud使用的解决方案是搭建配置中心将并指定一个配置文件路径如git项目对配置文件进行统一管理。 
在Spring Cloud中，提供了分布式配置中心组件spring cloud config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库中。
在实现cloud config中，主要有两个角色：作为配置中心连接配置路径的 config server,连接配置中心读取配置的config client。

1、引入依赖：
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

2、开启配置服务器
```
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

3、添加配置文件.yml
```
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false

server:
  port: 8040
spring:
  application:
    name: pig-config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/george93/pig-config.git  # 配置git仓库的地址
          #search-paths: config-repo # git仓库地址下的相对地址，可以配置多个，用,分割。
          username: # git仓库的账号
          password: # git仓库的密码
```

4、在需要从配置中心读取配置文件的服务中加入：
```
#是否从配置中心读取文件,配置中心的servieId，即服务名。
spring:
  cloud:
    config:
      discovery: 
        enabled: true
        serviceId: pig-config-server
```

### 对应用配置文件进行加密

为了防止配置文件被别人刻意访问或查看，可用加密手段进行处理，尤其是数据库密码，账号等：

#### 方式一
1、Config Server 加解密依赖JDK的JCE。 JDK8的下载:[地址](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)
```
curl localhost:4001/encrypt -d gqsu
密文
```

2、配置config serve encrypt.key=foo

3、使用config server 提供的加解密接口生成密文

4、配置文件使用密文：
```
spring:
  datasource:
    password: '{ciper}密文'

xxx: '{ciper}密文'   
```
5、其他的非对称加密等高级配置，[参考官方文档](http://cloud.spring.io/spring-cloud-static/Dalston.SR4/single/spring-cloud.html#_encryption_and_decryption_2)

#### 方式二
1、添加jasypt依赖：
```
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>1.16</version>
</dependency>
```

2、配置：
```
jasypt:
  encryptor:
    password: foo #根密码
```

3、调用JAVA API 生成密文
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = PigAdminApplication.class)
public class PigAdminApplicationTest {
	@Autowired
	private StringEncryptor stringEncryptor;

	@Test
	public void testEnvironmentProperties() {
		System.out.println(stringEncryptor.encrypt("gqsu"));
	}

}
```

4、在需要从配置中心读取配置文件的服务中加入：
```
spring:
  datasource:
    password: ENC(密文)

xxx: ENC(密文)
```

Spring Cloud Config 提供了统一的加解密方式，方便使用，
但是如果应用配置没有走配置中心，那么加解密过滤是无效的；依赖JCE 对于低版本spring cloud的兼容性不好。
jasypt 功能更为强大，支持的加密方式更多，但是如果多个微服务，需要每个服务模块引入依赖配置，较为麻烦；但是功能强大 、灵活。