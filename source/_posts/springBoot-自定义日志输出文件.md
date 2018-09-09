---
layout: post
title: springBoot-自定义日志输出文件
date: 2018-05-21 02:03:28
tags: springBoot
categories: springBoot
---

>默认情况下，Spring Boot会用Logback来记录日志，并用INFO级别输出到控制台

>springBoot自带的日志输出，对于一些错误sql不一定能打印出来。最好自己配置日志文件


参考： http://tengj.top/2017/04/05/springboot7/

在application.yml中配置：

```
logging:
  config: classpath:logback-spring.xml
```

在resource目录下，创建:logback-spring.xml。完整的 logback-spring.xml 文件：
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!--
        说明：
        1、日志级别及文件
            日志记录采用分级记录，级别与日志文件名对应，不同级别的日志信息记录到不同的日志文件中
            例如：error级别记录到log_error_xxx.log或log_error.log（该文件为当前记录的日志文件），而log_error_xxx.log为归档日志。
            日志文件按日期记录，同一天内，若日志文件大小等于或大于10M,则按0、1、2顺序分别命名
        2、Appender
            FILEERROR对应error级别，文件名以eduAppLog-error-xxx.log形式命名
            FILEWARN对应warn级别，文件名以eduAppLog-warn-xxx.log形式命名
            FILEINFO对应info级别，文件名以eduAppLog-info-xxx.log形式命名
            STDOUT将日志输出到控制台，方便测试使用
    -->


    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="log" />

    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>


    <!--日志记录器，日期滚动记录-->
    <appender name="FILEERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名-->
        <file>${LOG_HOME}/log_error.log</file>
        <!-- 日志记录器的滚动策略，按照日志，大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${LOG_HOME}/error/log-error-%d{yyyy-MM-dd}.%i.log</FileNamePattern>
            <maxFileSize>10MB</maxFileSize>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d %-5level %logger Line:%-3L - %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>error</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="FILEWARN" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名-->
        <file>${LOG_HOME}/log_warn.log</file>
        <!-- 日志记录器的滚动策略，按照日志，大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${LOG_HOME}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</FileNamePattern>
            <maxFileSize>10MB</maxFileSize>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d %-5level %logger Line:%-3L - %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>


    <appender name="FILEINFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名-->
        <file>${LOG_HOME}/log_info.log</file>
        <!-- 日志记录器的滚动策略，按照日志，大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${LOG_HOME}/info/log-info-%d{yyyy-MM-dd}.%i.log</FileNamePattern>
            <maxFileSize>10MB</maxFileSize>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d %-5level %logger Line:%-3L - %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 日志输出级别 -->
    <root level="info">
        <appender-ref ref="FILEERROR" />
        <appender-ref ref="FILEWARN" />
        <appender-ref ref="FILEINFO" />
        <appender-ref ref="STDOUT" />
    </root>

</configuration>

```