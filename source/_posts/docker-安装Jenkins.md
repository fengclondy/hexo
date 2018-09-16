---
layout: post
title: Docker上安装Jenkins
date: 2018-06-13 09:41:39
tags: docker
categoriets: docker
---

参考：https://blog.csdn.net/mmd0308/article/details/77206563?locationNum=6&fps=1

### 安装

1、拉取镜像
```powershell
$ docker pull jenkins
```
2、创建jenkins文件夹
```powershell
$ mkdir /home/project/jenkins
$ chown -R 1000:1000 jenkins/
```
>这里的权限一定要赋值为1000:1000，否则会报无权限操作。至于原因可能和docker配置有关（通过docker logs查看日志获得）。

![](http://p2jr3pegk.bkt.clouddn.com/jenkins01.png)

3、运行
```powershell
$ docker run -itd -p 8080:8080 -p 50000:50000 --name jenkins  -v /home/project/jenkins:/var/jenkins_home jenkins
```

4、查看启动状态及日志
```powershell
$ dockers ps
$ docker logs jenkins
```

5、登录8080端口使用页面，初次登录需要密码：
密码的获取地址为：
1.`cat /home/project/jenkins/secrets/initialAdminPassword`  
2.`docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`
