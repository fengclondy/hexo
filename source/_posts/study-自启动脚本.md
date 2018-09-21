---
layout: post
title: 自启动脚本学习
date: 2018-03-17 09:28:33
tags: study
categories: study
---


### Linux 自启动脚本创建

>Elasticsearch Linux 自启动脚本。

假设文件所在位置为/usr/local/el-start.sh。内容如下:

```powershell
#!/bin/bash
export JAVA_HOME=/usr/local/java/jdk1.8.0_121
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
su - elastic<<!
cd /home/elastic/elasticsearch-5.5.2/
./bin/elasticsearch -d
exit
!
```

<!-- more -->

#### 脚本解释说明

Elasticsearch启动需要jdk支持，故首先设置jdk系统变量
```powershell
export JAVA_HOME=/usr/local/java/jdk1.8.0_121
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

#### Elasticsearch启动
```powershell
#默认不支持root用户启动,切换用户
su - elastic

#进入Elasticsearch安装目录
cd /home/elastic/elasticsearch-5.5.2/

#以后台守护进程方式启动
./bin/elasticsearch -d

#脚本设置完成后执行脚本测试，如果启动成功。则可以将脚本加入系统文件中，系统重启就会自动加载。

#编辑文件
/etc/rc.d/rc.local

#加入
/usr/local/el-start.sh
```