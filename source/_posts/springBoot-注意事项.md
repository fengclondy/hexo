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

<!-- more -->

### SpringApplication的run方法执行过程
SpringApplication 对象的 run 方法的源码和运行流程。
```java
public ConfigurableApplicationContext run(String... args) {
    // 1、创建并启动计时监控类
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();

    // 2、初始化应用上下文和异常报告集合
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();

    // 3、设置系统属性 `java.awt.headless` 的值，默认值为：true
    configureHeadlessProperty();

    // 4、创建所有 Spring 运行监听器并发布应用启动事件
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();

    try {
        // 5、初始化默认应用参数类
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
                args);

        // 6、根据运行监听器和应用参数来准备 Spring 环境，读取配置
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                applicationArguments);
        configureIgnoreBeanInfo(environment);

        // 7、创建 Banner 打印类
        Banner printedBanner = printBanner(environment);

        // 8、创建应用上下文
        context = createApplicationContext();

        // 9、准备异常报告器
        exceptionReporters = getSpringFactoriesInstances(
                SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);

        // 10、准备应用上下文
        prepareContext(context, environment, listeners, applicationArguments,
                printedBanner);

        // 11、刷新应用上下文
        refreshContext(context);

        // 12、应用上下文刷新后置处理
        afterRefresh(context, applicationArguments);

        // 13、停止计时监控类
        stopWatch.stop();

        // 14、输出日志记录执行主类名、时间信息
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                    .logStarted(getApplicationLog(), stopWatch);
        }

        // 15、发布应用上下文启动完成事件
        listeners.started(context);

        // 16、执行所有 Runner 运行器
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        // 17、发布应用上下文就绪事件
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }

    // 18、返回应用上下文
    return context;
}
```