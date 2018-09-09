---
layout: post
title: docker入门知识
date: 2018-04-24 11:01:57
tags: docker
categoriets: docker
---

>问题产生：软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器跑起来？
用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有它们都正确，软件才能运行。举例来说，安装一个 Python 应用，计算机必须有 Python 引擎，还必须有各种依赖，可能还要配置环境变量。
如果某些老旧的模块与当前环境不兼容，那就麻烦了。开发者常常会说："它在我的机器可以跑了"（It works on my machine），言下之意就是，其他机器很可能跑不了。
环境配置如此麻烦，换一台机器，就要重来一次，旷日费时。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。

### 解决办法

#### 虚拟机

虚拟机（virtual machine）就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。
虽然用户可以通过虚拟机还原软件的原始环境。但是，这个方案有几个缺点。
- （1）资源占用多。  
虚拟机会独占一部分内存和硬盘空间。它运行的时候，其他程序就不能使用这些资源了。哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。
- （2）冗余步骤多。  
虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。
- （3）启动慢。  
启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行。

<!-- more -->

#### linux容器

由于虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Containers，缩写为 LXC）。
Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。或者说，在正常进程的外面套了一个保护层。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。
由于容器是进程级别的，相比虚拟机有很多优势。
- （1）启动快  
容器里面的应用，直接就是底层系统的一个进程，而不是虚拟机内部的进程。所以，启动容器相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多。
- （2）资源占用少  
容器只占用需要的资源，不占用那些没有用到的资源；虚拟机由于是完整的操作系统，不可避免要占用所有资源。另外，多个容器可以共享资源，虚拟机都是独享资源。
- （3）体积小  
容器只要包含用到的组件即可，而虚拟机是整个操作系统的打包，所以容器文件比虚拟机文件要小很多。  
总之，容器有点像轻量级的虚拟机，能够提供虚拟化的环境，但是成本开销小得多。


### 基本概念

Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。
Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

#### 用途

Docker 的主要用途，目前有三大类。
- （1）提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。  
- （2）提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。  
- （3）组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。  


#### docker的组成

Docker是CS架构，基本组成：

- Docker daemon: 

运行在宿主机上，Docker守护进程，用户通过Docker client(Docker命令)与Docker daemon交互

- Docker client: 

Docker 命令行工具，是用户使用Docker的主要方式，Docker client与Docker daemon通信并将结果返回给用户，
Docker client也可以通过socket或者RESTful api访问远程的Docker daemon

#### 重要概念

- Docker image：

镜像是只读的，镜像中包含有需要运行的文件。镜像用来创建container，一个镜像可以运行多个container；镜像可以通过Dockerfile创建，也可以从Docker hub/registry上下载。

- Docker container：

容器是Docker的运行组件，启动一个镜像就是一个容器，容器是一个隔离环境，多个容器之间不会相互影响，保证容器中的程序运行在一个相对安全的环境中。

- Docker hub/registry: 

共享和管理Docker镜像，用户可以上传或者下载上面的镜像，官方地址为https://registry.hub.docker.com/，也可以搭建自己私有的Docker registry。

### 深入理解 

镜像、容器、DockerFile三者之间的关系如下图所示：![](http://p2jr3pegk.bkt.clouddn.com/DockerFile.png)
>通过上图可以看出使用 Dockerfile 定义镜像，运行镜像启动容器。

#### Dockerfile 概念

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。有了 Dockerfile，当我们需要定制自己额外的需求时，只需在 Dockerfile 上添加或者修改指令，重新生成 image 即可，省去了敲命令的麻烦。


#### 构建镜像

docker build 命令会根据 Dockerfile 文件及上下文构建新 Docker 镜像。构建上下文是指 Dockerfile 所在的本地路径或一个URL（Git仓库地址）。构建上下文环境会被递归处理，所以构建所指定的路径还包括了子目录，而URL还包括了其中指定的子模块。

将当前目录做为构建上下文时，可以像下面这样使用docker build命令构建镜像：

```dockerfile
docker build .
Sending build context to Docker daemon  6.51 MB
...
```

>说明：构建会在 Docker 后台守护进程（daemon）中执行，而不是CLI中。构建前，构建进程会将全部内容（递归）发送到守护进程。大多情况下，应该将一个空目录作为构建上下文环境，并将 Dockerfile 文件放在该目录下。

在构建上下文中使用的 Dockerfile 文件，是一个构建指令文件。为了提高构建性能，可以通过.dockerignore文件排除上下文目录下不需要的文件和目录。

在 Docker 构建镜像的第一步，docker CLI 会先在上下文目录中寻找.dockerignore文件，根据.dockerignore 文件排除上下文目录中的部分文件和目录，然后把剩下的文件和目录传递给 Docker 服务。

Dockerfile 一般位于构建上下文的根目录下，也可以通过-f指定该文件的位置：

```dockerfile
docker build -f /path/to/a/Dockerfile .
```
>还可以通过-t参数指定构建成镜像的仓库、标签。


#### 镜像标签

```dockerfile
docker build -t nginx/v3 .
```

如果存在多个仓库下，或使用多个镜像标签，就可以使用多个-t参数：

```dockerfile
docker build -t nginx/v3:1.0.2 -t nginx/v3:latest .
```

注意，在 Docker 守护进程执行 Dockerfile 中的指令前，首先会对 Dockerfile 进行语法检查，有语法错误时会返回：

```dockerfile
docker build -t nginx/v3 .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```

#### 缓存

Docker 守护进程会一条一条的执行 Dockerfile 中的指令，而且会在每一步提交并生成一个新镜像，最后会输出最终镜像的ID。
生成完成后，Docker 守护进程会自动清理你发送的上下文。 Dockerfile文件中的每条指令会被独立执行，
并会创建一个新镜像，RUN cd /tmp等命令不会对下条指令产生影响。 Docker 会重用已生成的中间镜像，以加速docker build的构建速度。以下是一个使用了缓存镜像的执行过程：

```powershell
$ docker build -t svendowideit/ambassador .
Sending build context to Docker daemon 15.36 kB
Step 1/4 : FROM alpine:3.2
 ---> 31f630c65071
Step 2/4 : MAINTAINER SvenDowideit@home.org.au
 ---> Using cache
 ---> 2a1c91448f5f
Step 3/4 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
 ---> Using cache
 ---> 21ed6e7fbb73
Step 4/4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
 ---> Using cache
 ---> 7ea8aef582cc
Successfully built 7ea8aef582cc
```

构建缓存仅会使用本地父生成链上的镜像，如果不想使用本地缓存的镜像，也可以通过--cache-from指定缓存。指定后将不再使用本地生成的镜像链，而是从镜像仓库中下载。


#### 寻找缓存的逻辑

Docker 寻找缓存的逻辑其实就是树型结构根据 Dockerfile 指令遍历子节点的过程。下图可以说明这个逻辑。
```
  FROM base_image:version           Dockerfile:
           +----------+                FROM base_image:version
           |base image|                RUN cmd1  --> use cache because we found base image
           +-----X----+                RUN cmd11 --> use cache because we found cmd1
                / \
               /   \
       RUN cmd1     RUN cmd2           Dockerfile:
       +------+     +------+           FROM base_image:version
       |image1|     |image2|           RUN cmd2  --> use cache because we found base image
       +---X--+     +------+           RUN cmd21 --> not use cache because there's no child node
          / \                                        running cmd21, so we build a new image here
         /   \
RUN cmd11     RUN cmd12
+-------+     +-------+
|image11|     |image12|
+-------+     +-------+
```

大部分指令可以根据上述逻辑去寻找缓存，除了 ADD 和 COPY 。这两个指令会复制文件内容到镜像内，除了指令相同以外，Docker 还会检查每个文件内容校验(不包括最后修改时间和最后访问时间)，如果校验不一致，则不会使用缓存。

除了这两个命令，Docker 并不会去检查容器内的文件内容，比如 `RUN apt-get -y update`，每次执行时文件可能都不一样，但是 Docker 认为命令一致，会继续使用缓存。这样一来，以后构建时都不会再重新运行`apt-get -y update`。

如果 Docker 没有找到当前指令的缓存，则会构建一个新的镜像，并且之后的所有指令都不会再去寻找缓存。

摘自：http://www.ityouknow.com/docker/2018/03/12/docker-use-dockerfile.html


### 基本操作

>参考：https://blog.csdn.net/lanmei618/article/details/80109222

>centos 7以上系统安装

#### 安装、启动、测试

```powershell
yum install docker
```
```powershell
systemctl start docker.service
systemctl enable docker.service
```
```powershell
docker version
```

#### 配置

Docker 中国官方镜像加速可通过registry.docker-cn.com访问。该镜像库只包含流行的公有镜像，私有镜像仍需要从美国镜像库中拉取。

```
vi  /etc/docker/daemon.json
#添加后
{
    "registry-mirrors": ["https://registry.docker-cn.com"],
    "live-restore": true
}
```
可以采用阿里云的镜像配置。


#### 拉取docker镜像
```
docker pull image_name
```
#### 查看宿主机上的镜像，Docker镜像保存在/var/lib/docker目录下
```
docker images
```

#### 删除镜像
```
docker rmi  docker.io/tomcat:7.0.77-jre7   或者  docker rmi b39c68b7af30
```

#### 查看当前有哪些容器正在运行
```
docker ps
```

#### 查看所有容器
```
docker ps -a
```

#### 启动、停止、重启容器命令
```
docker start container_name/container_id
docker stop container_name/container_id
docker restart container_name/container_id
```

#### 后台启动一个容器后，如果想进入到这个容器，可以使用attach命令
```
docker attach container_name/container_id
```

#### 删除容器的命令
```
docker rm container_name/container_id
```

#### 删除所有停止的容器
```
docker rm $(docker ps -a -q)
```

#### 查看当前系统Docker信息
```
docker info
```

#### 从Docker hub上下载某个镜像
```
docker pull centos:latest
```
>执行docker pull centos会将Centos这个仓库下面的所有镜像下载到本地repository。

#### 查找Docker Hub上的nginx镜像
```
docker search nginx
```

以上操作仅做简单了解，后文有详细说明。这里以一个简单示例，作为展示：

- 启动测试镜像`docker run -d -p 91:80 kitematic/hello-world-nginx`

- 访问：http://localhost:91 测试，这里的localhost指的是宿主机的主机名
![](http://p2jr3pegk.bkt.clouddn.com/docker02-1.png)

