---
layout: post
title: linux-安装Jenkins，部署SpringBoot工程
date: 2018-07-23 07:07:39
tags:
---

### 安装准备
1、安装maven：
```
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
## 解压
tar vxf apache-maven-3.5.0-bin.tar.gz
## 移动
mv apache-maven-3.5.0 /usr/local/maven3
```
2、修改环境变量，在/etc/profile中添加：
```
MAVEN_HOME=/usr/local/maven3
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```
3、执行`source /etc/profile`使环境变量生效。然后`mvn -v`查看是否安装成功。

<!-- more -->

### 安装jenkins
1、下载安装包：
```
wget http://mirrors.jenkins.io/war/2.83/jenkins.war
```
2、启动服务(War包自带Jetty服务器)
```
java -jar jenkins.war &
```
3、第一次启动Jenkins时，会自动生成一个随机的口令。该口令用于，在打开`localhost:8080`后输入的密码。

默认的用户为：`admin`;
如果初始密码不记得了，请在`/root/.jenkins/secrets/initialAdminPassword`下查看

登陆成功后，可在控制台更改创建用户以及密码。

### 其他
1、关于重启和关闭,直接添加相应的关键字即可
```
#重启Jenkies
http://localhost:8080/restart
#关闭Jenkies
http://localhost:8080/exit
#重新加载配置信息
http://localhost:8080/reload

```

2、在系统管理-插件管理里安装一些推荐的插件，或者升级一些必要的插件。推荐安装`Git plugin`和`Maven Integration`，`publish over SSH`。

3、在系统管理-全局工具配置里，取消自动安装（不推荐使用自动，可手动安装），添加JDK和Maven环境依赖

4、配置 SSH免登陆，参考：[纯洁的微笑](http://www.ityouknow.com/springboot/2017/11/11/springboot-jenkins.html)

5、构建项目。选择新建任务，输入项目名称，选择maven项目,确定，然后选择`丢弃旧的构建`,自定义设置保持构建的天数和最大个数,源码管理选择git，




