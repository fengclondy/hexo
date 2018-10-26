---
layout: post
title: docker-compose.yml文件详解
date: 2018-06-09 06:11:41
tags: docker
categoriets: docker
---

参考博客：https://blog.csdn.net/qq_36148847/article/details/79427878

> 使用docker-compose进行统一构建，就不需要在写DockerFile文件了。

### 常见配置参数

#### version
版本号。有些命令属于高版本的，低版本执行不了。

#### build

指定 Dockerfile 所在文件夹的路径。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。
```yaml
build: /path/to/build/dir
```
注：该路径也可以是相对路径，如`build: ./dir`

<!-- more -->

#### command

覆盖容器启动后默认执行的命令。
```yaml
command: bundle exec thin -p 3000
```

#### container_name

为自定义的容器指定一个名称，而不是使用默认的名称
```yaml
container_name: my-web-container
```


#### links

链接到其它服务中的容器。使用服务名称（同时作为别名）或服务名称：服务别名 （SERVICE:ALIAS） 格式都可以。
```yaml
links:
- db
- db:database
- redis
```
使用的别名将会自动在服务容器中的 /etc/hosts 里创建。例如：`172.17.2.186 db`,相应的环境变量也将被创建。

#### external_links

链接到 docker-compose.yml 外部的容器，甚至 并非 Compose 管理的容器。参数格式跟 links 类似。
```yaml
external_links:
- redis_1
- project_db_1:mysql
- project_db_1:postgresql
```

#### ports

暴露端口信息。默认协议TCP,可以指明为UDP，使用 宿主：容器 （HOST:CONTAINER）格式或者仅仅指定容器的端口（宿主机将会随机选择端口）都可以。
```yaml
ports:
- "3000"
- "8000:8000"
- "127.0.0.1:8001:8001"
```
注：当使用 HOST:CONTAINER 格式来映射端口时，如果你使用的容器端口小于 60 你可能会得到错误得结果，因为 YAML 将会解析 xx:yy 这种数字格式为 60 进制。所以建议采用字符串格式。

#### expose

暴露端口，但不映射到宿主机，只被连接的服务访问。仅可以指定内部端口为参数
```yaml
expose:
- "3000"
- "8000"
```

#### volumes

卷挂载路径设置，映射到容器中。可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式 （HOST:CONTAINER:ro）。
格式：src:dest:mode（卷名：路径：权限）

- src：可以是卷名，宿主机目录等，可以是文件或者目录，若是文件则文件必须存在，否则会是目录的形式挂载。  
- dest：容器内的路径，可以是文件可以是目录，若是文件则文件必须存在，否则会是目录的形式挂载。  
- mode：权限，只读(ro)，可读可写(rw)  
```yaml
volumes:
- /var/lib/mysql
- cache/:/tmp/cache
- ~/configs:/etc/configs/:ro
```

#### volumes_from

从另一个服务或容器挂载它的所有卷。
```yaml
volumes_from:
- service_name
- container_name
```

#### depends_on

指明容器的依赖、启动的先后的顺序。
```yaml
depends_on:
 - nginx
 - mysql
```
注：如果存在依赖关系，需要用`docker-compose up service-name`以依赖顺序启动服务，默认会先启动对应的依赖容器


#### restart
什么情况下重新启动容器。默认值为 no ，即在任何情况下都不会重新启动容器；当值为 always 时，容器总是重新启动；
当值为 on-failure 时，当出现 on-failure 报错容器退出时，容器重新启动。
```yaml
restart: "no"
#restart: always
#restart: on-failure
#restart: unless-stopped
```


#### environment

设置环境变量(key=value)。你可以使用数组或字典两种格式。只给定名称的变量会自动获取它在 Compose 主机上的值，可以用来防止泄露不必要的数据。
```yaml
environment:
- RACK_ENV=development
- SESSION_SECRET
```

#### env_file

从文件中获取环境变量，可以为单独的文件路径或列表。
如果通过 `docker-compose -f FILE` 指定了模板文件，则 env_file 中路径会基于模板文件路径。
如果有变量名称与 environment 指令冲突，则以后者为准。
```yaml
env_file: .env
env_file:
- ./common.env
- ./apps/web.env
- /opt/secrets.env
```
环境变量文件中每一行必须符合格式，支持 # 开头的注释行。
```yaml
# common.env: Set Rails/Rack environment
RACK_ENV=development
```

#### extends

基于已有的服务进行扩展。例如我们已经有了一个 webapp 服务，模板文件为 common.yml。
```yaml
# common.yml
webapp:
build: ./webapp
environment:
\ - DEBUG=false
\ - SEND_EMAILS=false
```
编写一个新的 development.yml 文件，使用 common.yml 中的 webapp 服务进行扩展。

#### 附录

简单的例子如下：
```yaml
version: '3'
services:
  nginx:
   container_name: favorites-nginx
   image: nginx:1.13
   restart: always
   ports:
   - 80:80
   - 443:443
   volumes:
     - ./nginx/conf.d:/etc/nginx/conf.d
     - /tmp/logs:/var/log/nginx
     - /home/share:/home/share
     
  mysql:
   build: ./mysql
   environment:
     MYSQL_DATABASE: favorites
     MYSQL_ROOT_PASSWORD: root
     MYSQL_ROOT_HOST: '%'
     TZ: Asia/Shanghai
   ports:
   - "3306:3306"
   volumes:
     - ./mysql_data:/var/lib/mysql
   restart: always
  
  redis:
    image: redis
    links:
      - app

  app:
    restart: always
    build: ./app
    working_dir: /app
    volumes:
      - ./app:/app
      - ~/.m2:/root/.m2
      - /tmp/logs:/usr/local/logs
    expose:
      - "8080"
    command: mvn clean spring-boot:run -Drun.profiles=docker
    depends_on:
      - nginx
      - mysql

```



#### 进阶
描述：某docker-compose里有两个容器：db，web。都在同一个内部网络172.66.1.0/24网段里，其中容器web的网络接口为:172.66.1.100，而容器db的网络接口为:172.66.1.200。
```yaml
# docker-compose.yml 文件
version: '3'
networks:
  study_net:
    ipam:
      driver: default
        config:
          - subnet: 172.66.1.0/24
services:
  web:
    networks:
      study_net:
        ipv4_address: 172.66.1.100
  db:
    networks:
      study_net:
        ipv4_address: 172.66.1.200
```