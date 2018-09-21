---
layout: post
title: springBoot学习(1)
date: 2018-01-14 06:13:43
tags: springBoot
categories: springBoot
---

##### 理论基础
1、能够独立运行

Spring Boot创建的程序最终会打成一个jar包，因此，只需要java -jar your-project-name.jar一句命令就可以运行起来了。

2、内置Servlet容器

Spring Boot项目会内置一个容器，如Tomcat、Jetty等，这样我们就不需要再额外配置服务器去部署了。

3、自动装配

从spring最原始的xml配置bean一路走来，曾几何时觉得@Autowired自动装配已经非常好用了，而Spring Boot则可以根据类路径中的jar包、类，为jar包里的类自动装配bean，这样就更进一步地减少了配置的工作量。但是嘛，自动的总会出问题的，它并不能帮你解决任何问题，当你发现它做的不够的时候，还是需要手动地去配置一下，但是相比而言，它也能够应付一部分场景了。

4、摆脱繁杂的XML和代码配置

Spring 4.x里已经可以通过注解来实现配置了，而Spring Boot则把这一特性发挥到了极致，一个项目里你甚至可以不写一行xml，不写一行配置代码，就可以完成。

5、简化的Maven配置

Maven的出现已经可以帮我解决了很多依赖的问题，不用再一个一个jar包去下载导入，但是有没有这么一种感觉，我还是要在pom里面写很多，而且有时候我根本不知道要添加哪个jar包，这一点Spring Boot也想到了，它提供了一系列starter来简化Maven依赖，比如你只需要添加一个spring-boot-starter-web，如下，就可添加web开发中常用的依赖包。

<!-- more -->


##### 开始使用

1、（方法一）访问官网：https://start.spring.io/

在Spring Boot官方的项目生成页面，你只需要填写项目的信息，选择一下依赖，然后点击Generate Project就可以生成项目文件并下载zip包

2、（方法二）手动创建

* 1.创建空的Maven项目

* 2.添加Spring Boot父依赖，提供相关的默认依赖，指定版本号，如下；
```xml
<parent>
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>1.5.4.RELEASE</version>
  <relativePath/> <!-- lookup parent from repository -->
</parent>
​````

* 3.添加相关starter，比如这里添加了web支持和security支持,详细的可参见：
    [文档](https://qbgbook.gitbooks.io/spring-boot-reference-guide-zh/content/III.%20Using%20Spring%20Boot/13.5.%20Starters.html)

​```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

* 4.添加编译插件
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>	
    </plugin>
  </plugins>	
</build>
```
* 5.编写实例,在main中创建XXX.java文件完成创建
```java
package site.holten.springboot0;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
@SpringBootApplication
public class Springboot0Application {
    public static void main(String[] args) {
        SpringApplication.run(Springboot0Application.class, args);
    }
    @RequestMapping("/")
    public String homepage() {
        return "this is home page!";
    }
    @RequestMapping("/hello/{name}")
    public String sayHello(@PathVariable String name) {
        return "hello " + name;
    }
}
```
* 6.java run application，在浏览器中访问：http://localhost:8080/


##### 其他说明


* `SpringApplication.run(Springboot0Application.class, args)`：main方法中的这句话用来指定程序的入口；

* `@SpringBootApplication`：这个是最重要的，标识着这是一个Spring Boot项目，开启自动配置；

* `@RestController`：这个是Spring MVC里面的注解，组合了@Controller和@ResponseBody；

* `@RequestMapping`：用来映射请求路径/参数、处理类和方法；

* `@Controller`：表明这是一个Spring MVC的Controller，Dispatcher Servlet会自动扫描该类，发现其中的@RequestMapping注解并映射；

* `@ResponseBody`：用来支持把返回值放到response体内；


##### 推荐

学习教程：

[小白教程](http://blog.csdn.net/Peng_Hong_fu/article/details/53691705)
[围观大神操作](https://gitee.com/didispace/SpringBoot-Learning)
[其他](http://www.ityouknow.com/spring-boot.html)


##### Bug修复

1. 导入官网下载下来的zip包发现pom报错

将用户主目录下`.m2/repository/`下的依赖全部清空。然后，在eclipse中右键项目 `--> Maven --> Update Project`等待更新完成后，错误自动消失了。或者在IDEA中右击pom.xml，然后选择`Maven --> Reimport`即可。

2. @SpringBootApplication.java 首行 package 报错, package 打包时最后的文件名称没有输入，需手动输入。（这个java文件是整个项目的启动入口;扫描的时候是扫描该类的同包或者子包，否则会报404）

