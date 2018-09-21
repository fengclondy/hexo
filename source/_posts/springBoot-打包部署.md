---
layout: post
title: springBoot打包部署
date: 2018-02-13 15:02:48
tags: springBoot
categories: springBoot
---

>两种方式：一种是打包成 jar 包直接执行，另一种是打包成 war 包放到 tomcat 服务器下

>特别指出一点，bootstrap.yml 和 application.yml 不一样，bootstarp 会先加载进工程里，尤其体现在统一配置config 里面。

>如果同一个目录下，有application.yml也有application.properties，默认先读取application.properties。还有就是加载文件的顺序会覆盖。

### 打成jar包

1、方式一采用了maven管理：

```powershell
$ cd 项目跟目录（和pom.xml同级）
$ mvn clean package
## 或者执行下面的命令,排除测试代码后进行打包,即不进行测试
$ mvn clean package  -Dmaven.test.skip=true
## 不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下。
$ mvn package -DskipTests
```
>排除测试代码打包，还可以将`@SpringBootTest`等注释掉。

打包完成后jar包会生成到target目录下，命名一般是 项目名+版本号.jar 

启动jar包命令：

```powershell
$ java -jar  target/spring-boot-scheduler-1.0.0.jar
```

也可以使用后台运行的方式：

```powershell
$ nohup java -jar target/spring-boot-scheduler-1.0.0.jar &
```

其他可选择的启动配置：

```powershell
## 读取不同的配置文件
$ java -jar app.jar --spring.profiles.active=dev
## 设置jvm参数
$ java -Xms10m -Xmx80m -jar app.jar & 
## 指定端口号
$ java -jar app.jar --server.port=8080
## 引用外部配置文件(jar与xx.properties需在同一目录)
$ java -jar app.jar --Dspring.config.location=application.properties
## 指定启动分支
$ java -jar app.jar --spring.profiles.active = prod
```

<!-- more -->

2、方式二采用了gradle管理：

```powershell
gradle build
java -jar build/libs/mymodule-0.0.1-SNAPSHOT.jar
```

>备注：如何使用不同的IDEA工具可能需要使用到的打包工具。
生成Eclipse项目文件(产生.project、.classpath文件和target文件夹)：`mvn eclipse:eclipse`。
清除Eclipse工程：`mvn eclipse:clean`。

### 打成war包

#### 方式一采用 maven 管理：  

1、修改pom.xml文件：

```xml
<!--1.替换 jar 为 war-->
<!--<packaging>jar</packaging>-->
<packaging>war</packaging>
<!--2.打包时排除tomcat，scope属性设置为provided。-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
	<scope>provided</scope>
</dependency>
```

2、注册启动类：  

创建 ServletInitializer.java，继承 SpringBootServletInitializer ，覆盖 configure()，把启动类 Application 注册进去。
外部web应用服务器构建 Web Application Context 的时候，会把启动类添加进去。

```java
public class ServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
}
```

然后执行：

```powershell
$ mvn clean package  -Dmaven.test.skip=true
```

会在target目录下生成：项目名+版本号.war文件，拷贝到tomcat服务器中启动即可。

#### 方式二采用 gradle 管理：

如果使用的是gradle,基本步奏一样，build.gradle中添加war的支持，排除spring-boot-starter-tomcat：

```
...

apply plugin: 'war'

...

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.4.2.RELEASE"){
    	exclude mymodule:"spring-boot-starter-tomcat"
    }
}
...
```

再使用构建命令：

```powershell
$ gradle build
```

war会生成在build\libs 目录下。


### 生产运维

#### 查看JVM参数的值

```powershell
$ jinfo -flags pid
```

实例结果：

```
-XX:CICompilerCount=3 -XX:InitialHeapSize=234881024 -XX:MaxHeapSize=3743416320 -XX:MaxNewSize=1247805440 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=78118912 -XX:OldSize=156762112 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseParallelGC
```
- `XX:CICompilerCount` ：最大的并行编译数
- `XX:InitialHeapSize` 和 `-XX:MaxHeapSize` ：指定JVM的初始和最大堆内存大小
- `XX:MaxNewSize` ： JVM堆区域新生代内存的最大可分配大小
- `XX:+UseParallelGC` ：垃圾回收使用Parallel收集器

#### 重启运行  

方式1：简单粗暴（不推荐）

```
$ ps -ef|grep java 
##拿到对于Java程序的pid
$ kill -9 pid
## 再次重启
$ Java -jar  xxxx.jar
```

方式2：（推荐）  

若为 maven 方式，修改配置依赖

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

若为 grade 方式，修改为：

```
springBoot {
    executable = true
}
```

（1）直接通过 ./yourapp.jar 来启动.
（2）注册为服务。做一个软链接指向你的jar包并加入到init.d中，然后用命令来启动。
示例：

```
$ ln -s /var/yourapp/yourapp.jar /etc/init.d/yourapp
$ chmod +x /etc/init.d/yourapp
```

使用stop或者是restart命令去管理你的应用

```
$ /etc/init.d/yourapp start|stop|restart
## 或者 service yourapp start|stop|restart
```