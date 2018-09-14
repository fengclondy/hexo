---
layout: post
title: springBoot学习(2)
date: 2018-01-14 09:02:40
tags: springBoot
categories: springBoot
---

### 构建可执行 JAR
通常有两种选择：

* 1. 在 Eclipse 中运行 Maven 构建，`Run As > Maven Build`。在 Goals 文本字段中，
    输入 clean 和 package，然后单击 Run 按钮。当看到文本 “App Started” 时，就可以体验该应用程序了

* 2. 从命令行运行 Maven 构建 `mvn clean package`

### 运行可执行 JAR

- 进入项目根目录，执行：`java -jar target/spring-boot-demo01.jar`

其中 target 是构建的默认输出目录。启动成功后,就可以访问了。

- 默认启动端口为8080，你也可以手动设置，指定端口号: 

`java -jar spring-boot-demo01.jar --server.port=9090`

- 如果出现中文乱码,请添加上该字段 `-Dfile.encoding=utf=8`:

`java -Dfile.encoding=utf=8 -jar spring-boot-demo01.jar --server.port=9090`

- 引用外部配置文件，这里将jar与配置.properties放在同一个目录下

`java -jar spring-boot-demo01.jar --Dspring.config.location=application.properties`

- 如果在配置中指定了某个分支 spring.profiles.active = dev，也可以在运行时指定

`java -jar spring-boot-demo01.jar --spring.profiles.active = prod`

<!-- more -->

### SpringBoot使用数据库，初始化数据

#### 方式一：使用Jpa  

在使用`spring boot jpa`的情况下设置`spring.jpa.hibernate.ddl-auto`的属性设置为 `create` or `create-drop`的时候，
spring boot 启动时默认会扫描classpath下面（项目中一般是resources目录）是否有`import.sql`，如果有机会执行`import.sql`脚本。

#### 方式二：使用Spring JDBC  

在属性文件里加入：

```
spring:
    datasource:
      schema: database/data.sql  //脚本的路径
      sql-script-encoding: utf-8  //脚本的编码
    jpa:
      hibernate:
        ddl-auto: none
```

说明：ddl-auto 四个值的含义:
- `create`： 每次加载hibernate时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，哪怕两次没有任何改变也要这样执行，这就是导致数据库表数据丢失的一个重要原因。

- `create-drop` ：每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。

- `update`：最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据 model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等 应用第一次运行起来后才会。

- `validate` ：每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。 

- `none`: 什么都不做。

>区别：第一种方式启动的时候Jpa会自动创建表，import.sql只负责创建表单后的初始化数据。
第二种方式启动的时候不会创建表，需要在初始化脚本中判断表是否存在，再初始化脚本的步骤。

### SpringBoot使用校验框架validation校验(校验前台的传值)

* 1. bean 中添加标签

常用标签归纳：
```powershell
@Null	   限制只能为null
@NotNull	   限制必须不为null
@AssertFalse	  限制必须为false
@AssertTrue	   限制必须为true
@DecimalMax(value)	   限制必须为一个不大于指定值的数字
@DecimalMin(value)	   限制必须为一个不小于指定值的数字
@Digits(integer,fraction)	   限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction
@Future	   限制必须是一个将来的日期
@Max(value)	    限制必须为一个不大于指定值的数字
@Min(value)	     限制必须为一个不小于指定值的数字
@Past	   限制必须是一个过去的日期
@Pattern(value)	    限制必须符合指定的正则表达式
@Size(max,min)	    限制字符长度必须在min到max之间
@Past	  验证注解的元素值（日期类型）比当前时间早
@NotEmpty	   验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0）
@NotBlank	   验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的空格
@Email	    验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式
```

* 2. Controller中开启验证

* 3. resource 下新建错误信息配置文件
  
  在resource 目录下新建提示信息配置文件"ValidationMessages.properties"(名字不可更改)，
 ```xml
 user.name.notBlank=\u7528\u6237\u540d\u4e0d\u80fd\u4e3a\u7a7a
 ```
* 4. 自定义异常处理器，捕获错误信息
当验证不通过时会抛异常出来，异常的message 就是 ValidationMessages.properties 中配置的提示信息。


### 项目编写中常需要注意的问题

* 1.修改了实体类中的属性时（这里指的是更改名称），会在数据库中自动加上一列表示，
    之前的属性列并不会删除，需手动删除。要是更改了类型，则也许手动去数据库操作。
    
* 2.更新操作，需要和创建曹组属性传值一致，否则会出现nul

* 3.Date时间格式转换。

 数据库查询出来的日期，不想转换，可以在apllication.property中加入：
 ```xml
 spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
 spring.jackson.time-zone=GMT+8
 ```

 前台传过来的是String字符的时间，不想转换，可在实体类的date属性头上加入：
 ```xml
 @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
 ```
