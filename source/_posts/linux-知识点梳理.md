---
layout: post
title: linux学习知识点梳理
date: 2018-01-30 05:54:12
tags: linux
categories: linux
---

服务器系统：centOs 7

### 防火墙相关

注意：Centos 7使用firewalld代替了原来的iptables。所以7以前版本的iptables是不能用的。

#### 关闭防火墙

```powershell
$ systemctl stop firewalld.service             #停止firewall
$ systemctl disable firewalld.service        #禁止firewall开机启动
```

#### 开放端口

```powershell
$ firewall-cmd --add-port=80/tcp --permanent   #永久添加80端口
$ firewall-cmd --reload                          #重新载入配置
```

#### 查看端口占用情况

```powershell
$ lsof -i:80
```

<!-- more -->

#### 常用命令介绍

```powershell
$ firewall-cmd --state                           ##查看防火墙状态，是否是running
$ firewall-cmd --get-zones                       ##列出支持的zone
$ firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
$ firewall-cmd --query-service ftp               ##查看ftp服务是否支持，返回yes或者no
$ firewall-cmd --add-service=ftp                 ##临时开放ftp服务
$ firewall-cmd --add-service=ftp --permanent     ##永久开放ftp服务
$ firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
$ iptables -L -n                                 ##查看规则，这个命令是和iptables的相同的
$ firewall-cmd --help                            ##查看帮助文件
```


### 上传/下载指令相关

#### 安装 rz/sz

```powershell
$ cd /tmp                   #进入保存下载文件的路径，可自定义
$ wget http://www.ohse.de/uwe/releases/lrzsz-0.12.20.tar.gz   #下载文件
$ tar zxvf lrzsz-0.12.20.tar.gz                    #解压
$ cd lrzsz-0.12.20
$ ./configure && make && make install                #安装
```

安装过程默认把lsz和lrz安装到了/usr/local/bin/目录下，
现在我们并不能直接使用，下面创建软链接，并命名为rz/sz

```powershell
$ cd /usr/bin
$ sudo ln -s /usr/local/bin/lrz rz
$ sudo ln -s /usr/local/bin/lsz sz
```

安装yum：

```powershell
yum install -y lrzsz
```


### 网络设置与域名绑定

#### 静态Ip

说明：CentOS 7.0默认安装好之后是没有自动开启网络连接的！

```powershell
$ cd  /etc/sysconfig/network-scripts/  #进入网络配置文件目录
$ vi  ifcfg-eth0  #编辑配置文件，添加修改以下内容(可能是ifcfg-ensxx)
```

参考如下示例，进行修改：

```powershell
 DEVICE=eth0    #设备名称
 HWADDR=00:0C:29:8D:24:73   #mac地址
 TYPE=Ethernet    #网络类型
 BOOTPROTO=static  #启用静态IP地址
 DEFROUTE=yes
 PEERDNS=yes
 PEERROUTES=yes
 IPV4_FAILURE_FATAL=no
 IPV6INIT=yes
 IPV6_AUTOCONF=yes
 IPV6_DEFROUTE=yes
 IPV6_PEERDNS=yes
 IPV6_PEERROUTES=yes
 IPV6_FAILURE_FATAL=no
 NAME=eno16777736
 UUID=ae0965e7-22b9-45aa-8ec9-3f0a20a85d11
 ONBOOT=yes     #开机自启动(就是这里)
 IPADDR0=210.28.188.98     #设置静态IP地址
 PREFIXO0=24    #设置子网掩码
 GATEWAY0=210.28.188.1    #设置网关
 DNS1=8.8.8.8    #设置主DNS
 DNS2=8.8.4.4    #设置备DNS
```

 编辑完成后,`:wq!`保存，然后退出

 ```powershell
 $ service network restart   #重启网络（或者/etc/init.d/network restart）
 $ ping www.baidu.com  #测试网络是否正常
 ```

#### 设置主机域名www

```powershell
$ vi /etc/hostname #编辑配置文件
```

修改 localhost.localdomain 为 www

```powershell
$ vi /etc/hosts #编辑配置文件
```

修改 localhost.localdomain 为 www。最后一步,

```powershell
$ shutdown -r now  #重启系统
```