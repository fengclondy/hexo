---
layout: post
title: springBoot-整合MongoDB
date: 2018-09-04 08:46:27
tags: springBoot
categories: springBoot
---

### 配置文件优先级
1. file:./config/ (当前项目路径config目录下); 
2. file:./ (当前项目路径下); 
3. classpath:/config/ (类路径config目录下); 
4. classpath:/ (类路径config下).

优先级由高到底，高优先级的配置会覆盖低优先级的配置

### springBoot启动顺序
这里调用的是AbstractApplicationContext的refresh()方法刷新上下文。

- 1、this.prepareRefresh();
准备启动spring容器，设置容器的启动日期和活动标志

- 2、ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
主要是创建beanFactory，同时加载配置文件.xml中的beanDefinition  
通过String[] configLocations = getConfigLocations()获取资源路径，然后加载beanDefinition  

- 3、this.prepareBeanFactory(beanFactory);
给beanFactory注册一些标准组件，如ClassLoader，StandardEnvironment，BeanProcess  

- 4、this.postProcessBeanFactory(beanFactory);
提供给子类实现一些postProcess的注册，如AbstractRefreshableWebApplicationContext注册一些Servlet相关的postProcess，真对web进行生命周期管理的Scope，通过registerResolvableDependency()方法注册指定ServletRequest，HttpSession，WebRequest对象的工厂方法。

- 5、this.invokeBeanFactoryPostProcessors(beanFactory);
调用所有BeanFactoryProcessor的postProcessBeanFactory()方法  

- 6、this.registerBeanPostProcessors(beanFactory);
注册BeanPostProcessor，BeanPostProcessor作用是用于拦截Bean的创建  

- 7、this.initMessageSource();
初始化消息Bean  

- 8、this.initApplicationEventMulticaster();
初始化上下文的事件多播组建，ApplicationEvent触发时由multicaster通知给ApplicationListener  

- 9、this.onRefresh();
ApplicationContext初始化一些特殊的bean 

- 10、this.registerListeners();
注册事件监听器，事件监听Bean统一注册到multicaster里头，ApplicationEvent事件触发后会由multicaster广播  

- 11、this.finishBeanFactoryInitialization(beanFactory);
非延迟加载的单例Bean实例化

- 12、this.finishRefresh();
结束刷新


### SpringBoot调整Configuration的执行顺序
```java
@Configuration
@AutoConfigureBefore(BConfiguration.class)
public class AConfiguration {
 
    @Bean
    @ConditionalOnMissingBean(XXX.class)
    public XXX XXX() {
        return new XXX();
    }
 
    @Bean
    @ConditionalOnMissingBean(YYY.class)
    public YYY YYY() {
        return new YYY();
    }
 
}
```