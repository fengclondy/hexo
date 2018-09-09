---
layout: post
title: linux-搭建maven
date: 2018-08-07 01:31:36
tags:
---

### 安装
1、下载与配置
```
wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz
tar -zxvf apache-maven-3.2.5-bin.tar.gz

export MAVEN_HOME=/usr/local/apache-maven-3.2.5
export PATH=${PATH}:${MAVEN_HOME}/bin

mvn -v
```