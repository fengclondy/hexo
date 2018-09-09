---
layout: post
title: rabbitmq--安装与使用
date: 2018-06-23 09:38:05
tags: rabbitmq
categories: rabbitmq
---

### 安装

>由于RabbitMQ依赖Erlang， 所以需要先安装Erlang。

1、下载并安装 Erlang。

2、下载并安装 RabbitMQ Server。

开启web管理控制台:

```powershell
$ cd ~/sbin
$ rabbitmq-plugins enable rabbitmq_management
```
然后通过浏览器访问：http://localhost:15672.(默认用户名：guest，密码：guest)
![](http://p2jr3pegk.bkt.clouddn.com/rabbit-01.png)
![](http://p2jr3pegk.bkt.clouddn.com/rabbit-02.png)

<!-- more -->

### 一些基本命令

```powershell
$ chkconfig rabbitmq-server on  # 添加开机启动RabbitMQ服务
$ cd ~/sbin/service
$ rabbitmq-server start # 启动服务
$ rabbitmq-server status  # 查看服务状态
$ rabbitmq-server stop   # 停止服务
```

#### 查看当前所有用户
```powershell
$ rabbitmqctl list_users
```

#### 查看默认guest用户的权限
```powershell
$ rabbitmqctl list_user_permissions guest
```

#### 由于RabbitMQ默认的账号用户名和密码都是guest。为了安全起见, 先删掉默认用户
```powershell
$ rabbitmqctl delete_user guest
```

#### 添加新用户
```powershell
$ rabbitmqctl add_user username password
```

#### 设置用户tag
```powershell
$ rabbitmqctl set_user_tags username administrator
```

#### 赋予用户默认vhost的全部操作权限
```powershell
$ rabbitmqctl set_permissions -p / username ".*" ".*" ".*"
```

#### 查看用户的权限
```powershell
$ rabbitmqctl list_user_permissions username
```

#### 查看服务状态
```powershell
$ cd ~/sbin/service 
$ rabbitmq-server status  # 查看服务状态
```

### 开启用户远程访问

默认情况下，RabbitMQ的默认的guest用户只允许本机访问， 如果想让guest用户能够远程访问的话，只需要将配置文件中的loopback_users列表置为空即可，如下：
``{loopback_users, []}``

另外关于新添加的用户，直接就可以从远程访问的，如果想让新添加的用户只能本地访问，可以将用户名添加到上面的列表, 如只允许admin用户本机访问。
``{loopback_users, ["admin"]}``

更新配置后，别忘了重启服务哦！

可能还需要开启端口：
```
/sbin/iptables -I INPUT -p tcp --dport 5672 -j ACCEPT  
/sbin/iptables -I INPUT -p tcp --dport 15672 -j ACCEPT  
```


