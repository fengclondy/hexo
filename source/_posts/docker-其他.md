---
layout: post
title: docker--其他
date: 2018-10-24 11:01:57
tags: docker
categoriets: docker
---

### docker容器时间与主机时间不一致
可以分别通过命令：`date` 。查看当前主机和容器的时间是否一致，一般来说会差8个时区。
- CST应该是指（China Shanghai Time，东八区时间） 
- UTC应该是指（Coordinated Universal Time，标准时间） 

最简单的方式就是，直接复制主机的 /etc/localtime 到容器中
```powershell
docker cp /etc/localtime:container-name/container-id /etc/localtime
```

当然，在打包阶段你也可以事先指定，这样：
```powershell
#设置时区
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo 'Asia/Shanghai' >/etc/timezone \
```

或者这样：
```powershell
 volumes:
     - /etc/localtime:/etc/localtime
```
