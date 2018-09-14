---
layout: post
title: springBoot-注意事项
date: 2018-09-04 08:46:27
tags: springBoot
categories: springBoot
---


### springBoot资源位置的存放顺序
默认的读取文件位置：
-  classpath:/META-INF/resources/
-  classpath:/resources/
-  classpath:/static/
-  classpath:/public/
-  classpath:/myFile/

### springBoot启动顺序
1、通过 SpringFactoriesLoader 加载 META-INF/spring.factories 文件，获取并创建 SpringApplicationRunListener 对象
2、然后由 SpringApplicationRunListener 来发出 starting 消息
3、创建参数，并配置当前 SpringBoot 应用将要使用的 Environment
4、完成之后，依然由 SpringApplicationRunListener 来发出 environmentPrepared 消息
5、创建 ApplicationContext
6、初始化 ApplicationContext，并设置 Environment，加载相关配置等
7、由 SpringApplicationRunListener 来发出 contextPrepared 消息，告知SpringBoot 应用使用的 ApplicationContext 已准备OK
8、将各种 beans 装载入 ApplicationContext，继续由 SpringApplicationRunListener 来发出 contextLoaded 消息，告知 SpringBoot 应用使用的 ApplicationContext 已装填OK
9、refresh ApplicationContext，完成IoC容器可用的最后一步
10、由 SpringApplicationRunListener 来发出 started 消息
11、完成最终的程序的启动
12、由 SpringApplicationRunListener 来发出 running 消息，告知程序已运行起来了
