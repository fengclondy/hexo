---
layout: post
title: springCloud--注册中心Eureka
date: 2018-04-21 07:34:04
tags: springCloud
categories: springCloud
---


### Eureka

1、Eureka Server  提供服务注册和发现

2、Service Provider  服务提供方

将自身服务注册到Eureka，从而使服务消费方能够找到

3、Service Consumer 服务消费方

从Eureka获取注册服务列表，从而能够消费服务

### 实战

#### Eureka Server创建（服务注册中心）

1. 添加依赖：

```
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka-server</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

2. 添加启动代码中添加@EnableEurekaServer注解

```
@SpringBootApplication
@EnableEurekaServer
public class EurekaServiceRegisterApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServiceRegisterApplication.class, args);
	}
}
```

3. 配置文件

```
spring.application.name=spring-cloud-eureka

server.port=8000
#是否将自己注册到Eureka Server，默认为true
eureka.client.register-with-eureka=false
#是否从Eureka Server获取注册信息，默认为true
eureka.client.fetch-registry=false

#设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址
eureka.client.serviceUrl.defaultZone=http://192.168.2.102:${server.port}/eureka/

logging.level.com.netflix.eureka=OFF
logging.level.com.netflix.discovery=OFF
```

安装完成：(可以看到下面的页面，其中还没有发现任何服务)
![](http://p2jr3pegk.bkt.clouddn.com/eureka01-1.png)

#### Eureka Provider（服务提供方）

- 1.添加依赖

```
<dependencies>
    <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
    </dependency>
    <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-eureka-server</artifactId>
         <scope>test</scope>
    </dependency>
</dependencies>
```

- 2. 添加启动代码中添加@EnableDiscoveryClient注解

```
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaServerProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerProducerApplication.class, args);
    }
}
```

- 3. 配置文件

```
#服务提供者的名字
spring.application.name=COMPUTE-SERVICE

#服务提供者的端口号
server.port=8888

#服务注册中心的地址
eureka.client.serviceUrl.defaultZone=http://192.168.2.102:8761/eureka/
```

- 4. 编写服务实现类,向外提供服务

```
@RestController
public class ServiceInstanceRestController {

    private static final Logger logger = LoggerFactory.getLogger(ServiceInstanceRestController.class);

    @Autowired
    private DiscoveryClient discoveryClient; //服务发现客户端

    @GetMapping(value = "/add")
    public Integer add(@RequestParam Integer a, @RequestParam Integer b) {
        ServiceInstance instance = discoveryClient.getLocalServiceInstance();
        Integer r = a + b;
        logger.info("/add, host:" + instance.getHost() + ", service_id:" + instance.getServiceId() + ", result:" + r);
        return r;
    }
}
```

启动项目，完成后，可以在浏览器中显示：![](http://p2jr3pegk.bkt.clouddn.com/eureka01-4.png)

#### Eureka Provider（服务提供方）
>这里介绍两种实现方式：Feign方式和Ribbon方式。

一、Feign实现方式

- 1. 添加依赖

```
<dependencies>
        <!-- Feign实现声明式HTTP客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
        </dependency>
        <!-- eureka客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <!-- spring boot实现Java Web服务-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
</dependencies>
```

- 2. 启动类代码：

```
@EnableDiscoveryClient
@EnableFeignClients
@SpringBootApplication
public class EurekaConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerApplication.class);
    }
}
```

- 3. 添加配置

```
#应用名称
spring.application.name=feign-consumer

#端口号
server.port=9001

#注册中心的地址
eureka.client.serviceUrl.defaultZone=http://192.168.2.102:8761/eureka/
```

- 4. 编写服务

```
//web入口函数
@RestController
public class ConsumerController {

    @Autowired
    private ComputeClient computeClient;

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public Integer add() {
        return computeClient.add(10, 20);
    }
}
```

```
//实体类（接口实现）
@FeignClient("COMPUTE-SERVICE")
public interface ComputeClient {

    @RequestMapping(method = RequestMethod.GET, value = "/add")
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);
}
```

二、Ribbon实现方式

- 1. 添加依赖：

```
<dependencies>
        <!-- 客户端负载均衡 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>
        <!-- eureka客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <!-- spring boot实现Java Web服务-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

- 2. 启动类代码

```
@EnableDiscoveryClient //开启服务发现的能力
@SpringBootApplication
public class EurekaConsumerApplication {

    @Bean //定义REST客户端，RestTemplate实例
    @LoadBalanced //开启负债均衡的能力
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerApplication.class, args);
    }
}
```

- 3. 添加配置

```
#应用名称
spring.application.name=ribbon-consumer

#端口号
server.port=9000

#注册中心的地址
eureka.client.serviceUrl.defaultZone=http://192.168.2.102:8761/eureka/

```

- 4. 编写服务

```
@RestController
public class ConsumerController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping(value = "/add")
    public String add() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }
}
```

由于定义了不同的端口启动，所以可以同时启动这两个项目。启动完成，如图所示：
![](http://p2jr3pegk.bkt.clouddn.com/eureka01-2.png)


三、测试

除了在页面访问对应的端口完成测试外，也可以在后台控制台查看结果

启动fegin消费者，访问localhost:9000/add，也可以看到服务提供者已经收到了消费者发来的请求。
![](http://p2jr3pegk.bkt.clouddn.com/eureka01-5.png)


### 补充，集群搭建

1、创建application-peer1.properties，作为peer1服务中心的配置，并将serviceUrl指向peer2

```
spring.application.name=spring-cloud-eureka
server.port=8000
eureka.instance.hostname=peer1

eureka.client.serviceUrl.defaultZone=http://peer2:8001/eureka/
```

2、创建application-peer2.properties，作为peer2服务中心的配置，并将serviceUrl指向peer1

```
spring.application.name=spring-cloud-eureka
server.port=8001
eureka.instance.hostname=peer2

eureka.client.serviceUrl.defaultZone=http://peer1:8000/eureka/
```

3、host转换

```
127.0.0.1 peer1  
127.0.0.1 peer2  
```

4、打包启动

```
#打包
mvn clean package
# 分别以peer1和peeer2 配置信息启动eureka
java -jar spring-cloud-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer1
java -jar spring-cloud-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer2
```


