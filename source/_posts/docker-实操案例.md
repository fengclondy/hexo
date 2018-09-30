---
layout: post
title: docker实操小案例
date: 2018-06-02 06:17:58
tags: docker
categoriets: docker
---

### nginx服务小例子

1、从远程仓库拉取镜像文件nginx，`:latest`是标签，可以不加。（也可以直接从远程镜像仓库中拉取并运行镜像，即步骤2）。

```powershell
$ docker pull nginx：latest
```

2、启动一个镜像，`--name`起名为 webserver（可不加），并映射宿主机的 80 端口到容器的 80 端口。

```powershell
$ docker run --name webserver -d -p 80:80 nginx
```

>此时直接访问宿主机的IP地址，可以出现nginx欢迎界面

<!-- more -->

3、更改页面内容， 使用`docker exec`命令进入容器，并更改其内容。

```powershell
$ docker exec -it webserver bash 
root@3729b97e8226:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html 
root@3729b97e8226:/# exit
```

>可通过` docker diff webserver `查看前后操作发生了那些变化

4、保存修改的容器为镜像（实际情况下，尽量不要使用，不推荐；强烈建议使用 Dockerfile）。

```powershell
$ docker commit \    
>--author "Tao Wang <twang2218@gmail.com>" \    
>--message "修改了默认网页" \    
>webserver \    
>nginx:v2 
sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214
```

5、删除之前的并启动自己的容器。

```powershell
$ docker run --name webserver -d -p 80:80 nginx:v2
```

6、 比较不同版本的区别,使用`docker history`命令

```powershell
$ docker history nginx:v2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ccf2292989e7        4 minutes ago       nginx -g daemon off;                            163B                修改了默认网页
ae513a47849c        4 weeks ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  STOPSIGNAL [SIGTERM]         0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  EXPOSE 80/tcp                0B                  
<missing>           4 weeks ago         /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B                 
<missing>           4 weeks ago         /bin/sh -c set -x  && apt-get update  && apt…   53.7MB              
<missing>           4 weeks ago         /bin/sh -c #(nop)  ENV NJS_VERSION=1.13.12.0…   0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  ENV NGINX_VERSION=1.13.12…   0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop) ADD file:ec5be7eec56a74975…   55.3MB   
```

### mysql 服务启动

1、拉取

```powershell
$ docker pull mysql/mysql-server:latest
```

2、启动，并查看容器启动端口号

```powershell
$ docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql/mysql-server:latest  
$ docker ps
```

3、使用`exec`，进入容器内部

```powershell
$ docker exec -it container-id bash
```

4、操作Mysql

```powershell
$ mysql -u root -p
```

### redis 缓存--操作数据

1、查看redis有哪些键值对并进行相关操作

```powershell
$ docker run -it --name some-redis --link yl-redis --rm redis redis-cli -h yl-redis -p 6379
```
>格式：`docker run -it --name some-redis --link [容器名称] --rm [镜像名称] redis-cli -h [容器名称] -p 6379`

如果报错：![](http://p2jr3pegk.bkt.clouddn.com/docker05-1.png)

可以这样解决：`docker network ls`列出所有网络,选择名称对应的网络地址(此处为:yiliao-default).

![](http://p2jr3pegk.bkt.clouddn.com/docker05-2.png)

改写上面的代码,即可正确访问：
```
docker run -it --name some-redis --link yl-redis --net yiliao_default --rm redis redis-cli -h yl-redis -p 6379
```

2、如果安装的是docker非简洁版，可以使用`docker exec`命令来操作
```
docker exec -it [docker进程ID标识] bash
```
这样，就相当于进入了容器内部，操作与windows是一致的。


### 开源云项目--云收藏（笔记）

1、最新版的采用了docker部署，安装与部署详见README.md文件

2、修改了docker-compose.yml文件后，一定要先删除容器，然后在执行一遍`docker-compose up -d`。单纯的停止，在启动容器是不会生效的。
如：现在要增加容器内置的nginx的图片目录预览功能，修改docker-compose.yml,在nginx模块：

```dockerfile
 nginx:
   container_name: favorites-nginx
   image: nginx:1.13
   restart: always
   ports:
   - 80:80
   - 443:443
   volumes:    #劵
     - ./nginx/conf.d:/etc/nginx/conf.d
     - /tmp/logs:/var/log/nginx
     ###增加的行，前面是宿主机的地址，后面是映射到的容器地址。这样就可以操作宿主机实现容器的拷贝
     - /home/share:/home/share
    
```

更多案例请访问:[中文官方](https://docs.docker-cn.com/)





