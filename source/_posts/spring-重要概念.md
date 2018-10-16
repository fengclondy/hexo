---
layout: post
title: spring--重要概念
date: 2017-05-27 02:32:56
tags: spring
categoriets: spring
---

#### Spring Ioc反转控制

简单来说, IOC 解决了类与类之间的依赖关系，底层原理是使用的java的反射机制，最重要的当属Bean。
Bean中的核心组件有：BeanFactory、ApplicationContext、BeanDefinition。

一个常见的：
```JAVA
   ApplicationContext ctx = new FileSystemXmlApplicationContext
       ("spring-beans/src/test/resources/beans.xml");
   System.out.println("number : " + ctx.getBeanDefinitionCount());
   ((Person) ctx.getBean("person")).work();
 }
```

Ioc的初始化过程：
- 1、资源(Resource)定位;
- 2、BeanDefinition 的载入和 BeanFactory 的构造.
- 3、向 IOC 容器(BeanFactory)注册 BeanDefinition.
- 4、根据 lazy-init 属性初始化 Bean 实例和依赖注入.

<!-- more -->

### 事件与事件监听器

Java提供了实现事件监听机制的两个基础类：自定义事件类型扩展自 java.util.EventObject、事件的监听器扩展自 java.util.EventListener。