---
layout: post
title: netty-socketio-部署与运维常见问题
date: 2018-07-30 01:22:33
tags: netty-socketio
categories: netty-socketio
---

### netty-socketio的部署
由于采用了SpringBoot搭建，需要开启可以被访问的ip，这里的ip如果本地运行就是自己的ip或者localhost；

要想能够被局域网内的用户访问，则需改成ip地址；如果想被任何一个人访问，则需要公网的ip，这里我将其部署在了阿里云上，，

但是阿里云还需要设置。首先，需要在阿里云的后台开启需要的功能端口，然后将工程设置为socketio.server.host=0.0.0.0

进行打包（不运行测试）`mvn clean package -Dmaven.test.skip=true`，最后将jar包放置服务器上，后台进程运行即可。

### netty优化

```
http://www.52im.net/forum.php?mod=viewthread&tid=166&highlight=netty
```
