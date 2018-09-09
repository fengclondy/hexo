---
layout: post
title: nginx安装与配置
date: 2018-02-05 01:25:52
tags: nginx
categories: nginx
---

>这里的安装为 yum 安装，源码安装方式参见 linux标签 栏目
#### nginx安装
1、安装
```powershell
$ yum install nginx  
```

2、启动
```powershell
$ service nginx start  //也可以通过whereis nginx查看，默认启动项存放在第一个目录
```

3、测试
```powershell
$ wget http://127.0.0.1  
```
<!-- more -->

其实，nginx默认是开启的80端口，如果你的IP是公网的，可以直接在浏览器进行访问。
正常情况选，会出现如下界面：![](http://p2jr3pegk.bkt.clouddn.com/nginx01-1.png)

4、自定义，修改配置文件：
```powershell
$ nginx -t     //查看nginx配置路径
$ vi /etc/nginx/nginx.conf 
```

修改完成后，保存（由于是yum安装，可以直接调用系统服务service启动）：
```powershell
service nginx restart
```

#### 相关命令
```powershell
$ nginx -t   #验证配置是否正确

$ nginx -q   #测试配置时，只输出错误信息 

$ nginx -V   #查看Nginx的版本号

$ start nginx   #启动Nginx

$ nginx -s stop   #快速停止或关闭Nginx

$ nginx -s quit   #正常停止或关闭Nginx

$ nginx -s reload     #配置文件修改重装载命令
```