---
layout: post
title: linux环境搭建之安装JDK和Mysql
date: 2018-01-17 12:44:06
tags: linux
categories: linux
---

服务器系统：centOs 7
```powershell
注意版本号要与服务器对应
```

```powershell
安装前准备：已下载好对应的JDK和Mysql安装包
当然，你也可以通过wget +地址的方式进行在线下载
```

##### wget工具安装
```powershell
$ yum -y install wget
```
>在虚拟机中可能会出现安装失败，解决办法：
进入编辑模式`vi /etc/sysconfig/network-scripts/ifcfg-enp0s3` ，
将`ONBOOT=no `改为`ONBOOT=yes`，就OK。
保存后重启网卡： `service network restart`即可 

<!-- more -->

### JDK安装

去官网下载JDK安装包

1、进入` /usr/local/java`目录下（该目录即对应的环境安装目录）

2、解压压缩文件：
```powershell
$ tar -zxvf jdk-8u161-linux-x64.tar.gz
```

3、编辑配置文件，配置环境变量：
```vim
$ vim /etc/profile
```

在profile文件的末尾加入环境变量：
```vim
#java
JAVA_HOME=/usr/local/java/jdk1.8.0_151
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

4、使配置文件立即生效
```powershell
$ source /etc/profile
```

5、查看是否安装完成
```powershell
$ java -version
```


### Mysql安装

1、下载mysql源安装包

```powershell
$ wget http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
```
2、安装mysql源包

```powershell
$ yum install mysql57-community-release-el7-9.noarch.rpm
```

3、检查mysql源是否安装成功

```powershell
$ yum repolist enabled | grep "mysql.*-community.*"
```
或者使用`yum repolist all | grep mysql`命令  
正常安装会出现如下界面：![](http://p2jr3pegk.bkt.clouddn.com/linux01-1.png)

4、安装MySQL服务

```powershell
$  yum -y install mysql-community-server
```



下面参考：（外网链接阿里ECS mysql 3306端口）
https://segmentfault.com/q/1010000009603559?sort=created

5、将MySQL服务加入开机启动
```powershell
$ systemctl enable mysqld
```

6、启动mysql服务进程 
```powershell
$ systemctl start mysqld
```

7、配置mysql(新版本密码设置要求：大小写+数字+特殊字符)

```powershell
$ mysql_secure_installation
```

> 也可以通过命令：`ALTER USER 'root'@'localhost' IDENTIFIED BY 'xxxxxx'`; 执行后：`flush privileges` ,刷新权限。yum安装系统会默认生成随机密码，可用过`grep 'password' /var/log/mysqld.log |head -n1`查看。

8、设置mysql（外部不能访问mysql）

```powershell
$ mysql -u root -p
>use mysql;
>update user set host=’%’ where user=’root’;
>select host,user from user;
$ service mysqld restart
```

9、其他相关

```powershell
$ cat /var/log/mysqld.log       #查看mysql操作日志
$ service mysqld restart       #重启mysql服务
```

