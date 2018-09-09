---
layout: post
title: linux-安装MongoDB
date: 2018-08-06 09:52:38
tags:
---

### 下载与安装

```powershell
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.6.tgz    # 下载
tar -zxvf mongodb-linux-x86_64-3.0.6.tgz                                   # 解压

mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb                         # 将解压包拷贝到指定目录
export PATH=<mongodb-install-directory>/bin:$PATH    #添加path
mkdir -p /data/db    #创建数据库
./mongod    #运行服务
```

