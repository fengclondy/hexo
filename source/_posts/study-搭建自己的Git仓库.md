---
layout: post
title: 搭建自己的的git仓库
date: 2018-04-28 06:53:52
tags: study
categories: study
---

>首先你需要一个服务器作为仓库（centos 7），可被外网访问

### 搭建自己的的git仓库

#### 服务端

1、下载git

```
$ yum -y install git   
```

2、新建仓库，地址与名称任意

```
$ cd /usr/local  
$ mkdir git  
$ cd git  
$ git init --bare hello.git  
```

3、创建git用户（修改默认密码）并赋予hello仓库权限

```
$ useradd git      
$ passwd git  
$ chown -R git:git hello.git  
```

<!-- more -->

#### 客户端

1、创建用户

```
git config --global user.name "你的名字"  
git config --global user.email "你的邮箱"  
```

2、创建密钥(ssh-keygen.exe在git安装目录的~./usr/bin下)

```
ssh-keygen -t rsa -C "你的邮箱"  
```

3、将公钥添加至服务器

将`C:\Users\Administrator\.ssh\id_rsa.pub`内容拷贝到`/root/.ssh/authorized_keys`

4、克隆远程仓库

`git clone git@106.14.132.65.168:/usr/local/git/hello.git`

5、创建test.txt文件并提交

```
echo 'hello world'>test.txt  
git add test.txt  
git commit -m'测试'  
git push  
```

6、再次新建文件夹克隆，查看是否能够收到刚才的提交文件

