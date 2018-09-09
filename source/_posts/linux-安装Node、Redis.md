---
layout: post
title: linux环境搭建之Node、Redis
date: 2018-01-30 05:54:12
tags: linux
categories: linux
---

服务器系统：centOs 7


#### node.js部署之启动后台运行 forever

##### 安装

可官网下载安装包上传或者选择直接在线下载，默认存放位置为tmp
```powershell
$ cd /usr      
$ mkdir nodejs     #创建node存放目录
$ cp /tmp/node-v8.9.3-linux-x64.tar.xz /usr/nodejs
$ tar -xvf node-v8.9.3-linux-x64.tar.xz       #解压
```

建立软连接，变为全局：
```powershell
$ ln -s /usr/nodejs/node-v8.9.3-linux-x64/bin/npm /usr/local/bin/ 
$ ln -s /usr/nodejs/node-v8.9.3-linux-x64/bin/node /usr/local/bin/
```

查看是否安装完成：
```powershell
$ node -v
```

<!-- more -->

##### 安装forever全局进程

```powershell
$ npm install forever -g
```

##### 运行
```powershell
$ forever start XXX.js
```

如果你需要用npm start来运行你的程序，则用命令

```
forever start -c "npm start" 路径
```

##### 关闭
```powershell
$ forever stop XXX.js
```


#### 安装Reids

下载地址：[官网](http://redis.io/download)，下载最新版本。这里采用的是源码编译安装方式

```
$ wget http://download.redis.io/redis-stable.tar.gz
$ tar -zxvf redis-stable.tar.gz 
$ mv redis-stable /usr/redis
$ cd /usr/redis/
$ make
$ cd /src
$ make install
$ ./redis-server
```

##### 配置
编辑redis.conf配置文件，设置为后台启动，将属性 daemonize 属性改为yes




##### 配置redis集群
1、安装集群脚本ruby文件的处理工具：
```
##安装Ruby
$ yum install ruby
$ ruby -v
```
2、


