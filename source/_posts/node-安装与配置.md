---
layout: post
title: nodeJS安装与配置
date: 2018-01-21 02:05:37
tags: node
categories: node
---

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

##### 安装forever全局进程
```powershell
$ npm install forever -g
```

##### 运行
```powershell
$ forever start index.js
```

##### 关闭
```powershell
$ forever stop index.js
```

<!-- more -->