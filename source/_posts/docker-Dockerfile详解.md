---
layout: post
title: Dockerfile文件详解
date: 2018-05-09 02:43:09
tags: docker
categoriets: docker
---

#### Dockerfile 文件组成

Dockerfile 分为四部分：**基础镜像信息**、**维护者信息**、**镜像操作指令**、**容器启动执行指令**。

```dockerfile
##  Dockerfile文件格式

# This dockerfile uses the ubuntu image
# VERSION 2 - EDITION 1
# Author: docker_user
# Command format: Instruction [arguments / command] ..
 
# 1、第一行必须指定 基础镜像信息
FROM ubuntu
 
# 2、维护者信息
MAINTAINER docker_user docker_user@email.com
 
# 3、镜像操作指令
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
 
# 4、容器启动执行指令
CMD /usr/sbin/nginx
```

一开始必须要指明所基于的镜像名称，接下来一般会说明维护者信息；后面则是镜像操作指令，例如 RUN 指令。每执行一条RUN 指令，镜像添加新的一层，并提交；最后是 CMD 指令，来指明运行容器时的操作命令。

<!-- more -->


#### Dockerfile 常用指令

一个例子：

```dockerfile
##  Dockerfile文件格式

# This dockerfile uses the ubuntu image
# VERSION 2 - EDITION 1
# Author: docker_user
# Command format: Instruction [arguments / command] ..
 
# 1、第一行必须指定 基础镜像信息。可指定多个FROM
FROM ubuntu
 
# 2、维护者信息
MAINTAINER docker_user docker_user@email.com

（下面是可选参数）
#为镜像添加元数据说明
# LABEL version="1.0"
#为Docker容器设置对外的端口号
# EXPOSE 8888
# 设置环境变量JAVA_HOME
#ENV JAVA_HOME /path/to/java 
 
# 3、镜像操作指令
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
 
# 4、容器启动执行指令
CMD /usr/sbin/nginx
```

指令集合：`FROM、MAINTAINER、RUN、LABEL、EXPOSE、ENV、ADD、COPY、
ENTRYPOINT（启动时执行的命令）、VOLUME（持久化存储数据）、USER（启动容器的用户）、ARG（定义一个变量）`

参考:[博客](http://book.itmuch.com/3%20%E4%BD%BF%E7%94%A8Docker%E6%9E%84%E5%BB%BA%E5%BE%AE%E6%9C%8D%E5%8A%A1/3.5%20Docker%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93%E7%9A%84%E6%90%AD%E5%BB%BA%E4%B8%8E%E4%BD%BF%E7%94%A8.html)

#### 使用Dockerfile构建Docker镜像

1、使用Maven打包项目 `mvn clean package`，默认会在target目录下生成对应的jar包文件。

2、与jar包的同级目录，创建文件，命名为Dockerfile。

```dockerfile
# 基于哪个镜像
FROM java:8

# 将本地文件夹挂载到当前容器
VOLUME /tmp

# 拷贝文件到容器，也可以直接写成ADD microservice-discovery-eureka-0.0.1-SNAPSHOT.jar /app.jar
ADD microservice-discovery-eureka-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'

# 开放8761端口
EXPOSE 8761

# 配置容器启动后执行的命令
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

3、构建镜像，执行：

`docker build -t eacdy/test1 . `    ---- 格式：`docker build -t 标签名称 Dockerfile的相对位置`

4、启动镜像：`docker run -p 8761:8761 eacdy/test1`。

5、测试访问http://Docker宿主机IP:8761 ，看是否能正常访问。

##### 编写 Dockerfile 文件

1、新建.dockerignore 文件，添加（排除不需要打包文件）：

```
.git
node_modules
npm-debug.log
```

2、新建 DocketFile文件

```dockerfile
FROM node:8.4COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
```

```makefile
- FROM node:8.4：该 image 文件继承官方的 node image，冒号表示标签，这里标签是8.4，即8.4版本的 node。

- COPY . /app：将当前目录下的所有文件（除了.dockerignore排除的路径），都拷贝进入 image 文件的/app目录。

- WORKDIR /app：指定接下来的工作路径为/app。

- RUN npm install：在/app目录下，运行npm install命令安装依赖。注意，安装后所有的依赖，都将打包进入 image 文件。

- EXPOSE 3000：将容器 3000 端口暴露出来， 允许外部连接这个端口。
```


#### 使用Maven插件构建Docker镜像

