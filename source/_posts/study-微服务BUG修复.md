---
layout: post
title: 实战总结之BUG修复
date: 2018-07-23 02:22:44
tags: study
categories: study
---

### 请求参数

```
public AjaxResult getMsgQueue(@RequestBody JsonParam param)  //处理请求体是json格式的数据

public AjaxResult getMsgQueue(@RequestParam String name)  //处理请求体是form-data格式的数据
```

### 时间入库多1s

mysql中的时间格式是`java.sql.date`,而java代码里面的时间格式是`java.util.date`,这是两种不同的格式。
在用到Mybatis时，发现存入数据库的时间总是比前台传递来的时间轴要多一秒，原因是在入库时，数据库对时间格式进行
了四舍五入，将后面的毫秒进位了。有两种方式改变：改变入库的时间格式，值精确到秒；或者改变数据库字段，将datetime
类型的长度设置为3，即精确到毫秒。默认只精确到秒。


### 跨域问题总结

报错：Failed to load http://localhost:8081/socket.io/?EIO=3&transport=polling&t=1523265142408-2: The 'Access-Control-Allow-Origin' header has a value 'null' that is not equal to the supplied origin. 
Origin 'null' is therefore not allowed access.

>典型的跨域问题。跨域是指 不同域名之间相互访问。跨域，指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对JavaScript施加的安全限制。
也就是如果在A网站中，我们希望使用Ajax来获得B网站中的特定内容 
如果A网站与B网站不在同一个域中，那么就出现了跨域访问问题。

参考：https://blog.csdn.net/lenkvin/article/details/79482205

### Docker容器时间与主机时间不一致的问题

查看主机的时间：
```
[root@localhost ~]# date
2016年 07月 27日 星期三 22:42:44 CST
```
查看容器的时间：
```
root@b43340ecf5ef:/# date                                                                                                                                                                                                                                                    
Wed Jul 27 14:43:31 UTC 2016
```
CST应该是指（China Shanghai Time，东八区时间） 
UTC应该是指（Coordinated Universal Time，标准时间） 
所以，这2个时间实际上应该相差8个小时。

>解决办法：
1、可以复制主机的localtime，并重启mysql服务或者是docker服务
```
docker cp /etc/localtime [容器ID或者NAME]:/etc/localtime
```
2、可以创建自定义的dockerfile文件,并使用docker build重新构建项目
```
FROM redis
FROM tomcat
ENV CATALINA_HOME /usr/local/tomcat

#设置时区
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    && echo 'Asia/Shanghai' >/etc/timezone 
```

### 记一次偶然遇到mysql的max_connection_errors错误
>Host is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'

参考：https://www.cnblogs.com/kerrycode/p/8405862.html

原因：MySQL服务器已经从某个host接收了大量中途终止的连接，于是决定终止继续接收来自该host的连接，允许最大的连接错误数为max_connect_errors，通过show variables命令可以查询，一般为10。

>解决办法：
1、可以在数据库中直接执行：`flush hosts`;
2、可以手动更改数据库的最大连接出错设置,依次运行：`show variables like '%max_connection_errors%';`
`set global max_connect_errors = 1000;`

