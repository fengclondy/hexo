---
layout: post
title: linux--其他相关
date: 2018-06-11 05:30:20
tags: linux
categories: linux
---


### 安装rabbitmq

1、安装：
```powershell
#安装epel源[EPEL (Extra Packages for Enterprise Linux，企业版Linux的额外软件包) 是Fedora小组维护的一个软件仓库项目为RHEL/CentOS提供他们默认不提供的软件.]
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
# 扩展erlang源安装最新的erlang
rpm -Uvh http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
yum -y install erlang
# 安装RabbitMQ
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm
rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
yum install rabbitmq-server-3.6.1-1.noarch.rpm
```

<!-- more -->

2、配置：

```powershell
复制代码
# 启动服务
/etc/init.d/rabbitmq-server start
# 添加用户
rabbitmqctl add_user admin admin
# 添加管理员权限
rabbitmqctl set_user_tags admin administrator
# 修改密码 
abbitmqctl add_user admin youpassword
# 设置权限
rabbitmqctl  set_permissions  -p  '/'  admin '.' '.' '.'
```

3、启动web管理

```powershell
# 启动Web插件
rabbitmq-plugins enable rabbitmq_management
# 删除guest用户
rabbitmqctl delete_user guest
# 添加Web访问权限
"""注意：rabbitmq从3.3.0开始禁止使用guest/guest权限通过除localhost外的访问。如果想使用guest/guest通过远程机器访问，需要在rabbitmq配置文件中(/etc/rabbitmq/rabbitmq.config)中设置loopback_users为[],配置文件不存在创建即可。"""
# 添加配置
[{rabbit, [{loopback_users, ["admin"]}]}].
```

