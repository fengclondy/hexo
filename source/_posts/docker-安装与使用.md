---
layout: post
title: docker的安装与使用
date: 2018-05-09 10:39:44
tags: docker
categoriets: docker
---


### 安装

> 使用教程参考：[官网](http://docs.docker-cn.com)

#### 安装最新版 docker

```powershell
## 安装依赖
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2

## 安装国内阿里源，不推荐采用官方源，访问太慢
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
## 更新 yum 软件源缓存，并安装 docker-ce
$ sudo yum makecache fast
$ sudo yum install docker-ce

## 启动
$ sudo systemctl enable docker
$ sudo systemctl start docker

## （可选）默认情况下，只有 root 用户和 docker 组的用户才可以访问 Docker（不推荐直接用root）
$ useradd -g docker docker
$ passwd docker
$ su docker
$ docker --version
```

>如果需要最新版本的 Docker CE 请使用以下命令:

```powershell
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test
```

<!-- more -->

#### 安装 docker-compose（方便管理）

```powershell
### 方式1 ###
## 检查有没有，没有在执行下面的;否则直接2
$ pip -V
$ pip install --upgrade pip
## 步骤1
$ yum -y install epel-release
$ yum -y install python-pip
## 步骤2
$ pip install docker-compose
$ docker-compose -version


### 方式2 ###
$ curl -L https://github.com/docker/compose/releases/download/1.21.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```


### 使用

#### 镜像相关

##### 搜索镜像 docker search

命令格式：`docker search [OPTIONS] TERM`

可选参数：

```shell
--automated		  #只列出自动构建的镜像
--filter, -f		#根据指定条件过滤结果
--limit	25	     #搜索结果的最大条数(默认25)
--no-trunc		  #不截断输出，显示完整的输出
--stars, -s		只展示Star不低于该数值的结果(默认0)
```

##### 下载镜像 docker pull

命令格式：`docker pull [OPTIONS] NAME[:TAG|@DIGEST]`

可选参数：

```shell
--all-tags, -a	   #下载所有标签的镜像
--disable-content-trust		  #忽略镜像的校验(默认true)
```
![](http://p2jr3pegk.bkt.clouddn.com/docker03-2.png)

结果参数含义：

- `NAME`：镜像仓库名称。

- `DESCRIPTION`：镜像仓库描述。

- `STARS`：镜像仓库收藏数，表示该镜像仓库的受欢迎程度，类似于GitHub的Stars。

- `OFFICAL`：表示是否为官方仓库，该列标记为[OK]的镜像均由各软件的官方项目组创建和维护。由结果可知，java这个镜像仓库是官方仓库，而其他的仓库都不是镜像仓库。

- `AUTOMATED`：表示是否是自动构建的镜像仓库。


##### 列出镜像 docker images

命令格式：`docker images [OPTIONS] [REPOSITORY[:TAG]]`

可选参数：

```shell
--all, -a		 #列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）
--digests		 #显示摘要信息
--filter, -f		#显示满足条件的镜像
--format		 #通过Go语言模板文件展示镜像
--no-trunc		 #不截断输出，显示完整的镜像信息
--quiet, -q		 #只显示镜像ID
```

![](http://p2jr3pegk.bkt.clouddn.com/docker03-1.png)

结果参数含义：

- `REPOSITORY`：镜像所属仓库名称。

- `TAG`：镜像标签。默认是latest，表示最新。

- `IMAGE ID`：镜像ID，表示镜像唯一标识。

- `CREATED`：镜像创建时间。

- `SIZE`：镜像大小。


##### 删除本地镜像 docker rmi

命令格式：`docker rmi [OPTIONS] IMAGE [IMAGE...]`

可选参数：

```shell
--force, -f		#强制删除
--no-prune		#不移除该镜像的过程镜像，默认移除
```

##### 保存镜像 docker save

命令格式：`docker save [OPTIONS] IMAGE [IMAGE...]`

可选参数：
```shell
--output, -o	 #输出到外部文件
```

##### 加载镜像 docer load

命令格式：`docker load [OPTIONS]`

可选参数：

```shell
--input, -i		#从文件加载而非STDIN
--quiet, -q		 #静默加载
```

##### 构建镜像 docker build

命令格式：`docker build [OPTIONS] PATH | URL | -`

可选参数：

```shell
--add-host		 #添加自定义从host到IP的映射，格式为（host:ip）
--build-arg		#设置构建时的变量
--cache-from		#作为缓存源的镜像
--cgroup-parent		#容器可选的父cgroup
--compress	false	#使用gzip压缩构建上下文
--cpu-period		#限制CPU CFS (Completely Fair Scheduler) 周期
--cpu-quota	0	#限制CPU CFS (Completely Fair Scheduler) 配额
--cpu-shares, -c		#CPU使用权重（相对权重）
--cpuset-cpus		#指定允许执行的CPU
--cpuset-mems		#指定允许执行的内存
--disable-content-trust	true	#忽略校验
--file, -f		#指定Dockerfile的名称，默认是‘PATH/Dockerfile’
--force-rm	false	#删除中间容器
--iidfile		#将镜像ID写到文件中
--isolation		#容器隔离技术
--label		#设置镜像使用的元数据
--memory, -m		#设置内存限制
--memory-swap		#设置Swap的最大值为内存+swap，如果设置为-1表示不限swap
--network		#在构建期间设置RUN指令的网络模式
--no-cache		#构建镜像过程中不使用缓存
--pull	false	#总是尝试去更新镜像的新版本
--quiet, -q		#静默模式，构建成功后只输出镜像ID
--rm	true	#构建成功后立即删除中间容器
--security-opt		#安全选项
--shm-size		#指定/dev/shm 目录的大小
--squash		#将构建的层压缩成一个新的层
--tag, -t		#设置标签，格式：name:tag，tag可选
--target		#设置构建时的目标构建阶段
--ulimit		#Ulimit 选项
```


##### 运行镜像 docker run

`docker run --name container-name -d -p 8088:80 image-name`

参数说明：
```shell
-d                        # 后台运行
-p 宿主机端口:容器端口       # 开放容器端口到宿主机端口
```

#### 容器相关

##### 查看容器状态（运行/停止）

`docker ps -a`

###### 启动容器

`docker start container-name/container-id`

##### 停止容器

`docker stop container-name/container-id`

还有一种强制关闭进程的方法(不推荐)：

`docker kill 容器id`

##### 端口映射

`docker run -d -p 6378:6379 --name port-reids redis`

##### 删除容器

`docker rm container-id`

##### 容器日志

`docker logs container-name/container-id`

##### 容器端口
`docker port container-name/container-id`


### 其他

1、容器内部启动：
`docker exec -t -i container-name/container-id /bin/bash`

2、自动重启：
`--restart=always`  、 `--restart=on-failure:5`

3、更多容器
`docker inspect container-name/container-id`

4、一键查看docker内存占用和清理未运行的容器命令
`docker system df` 、 `docker system prune`



