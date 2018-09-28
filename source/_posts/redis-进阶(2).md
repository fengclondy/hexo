---
layout: post
title: redis进阶(2)
date: 2018-07-30 01:20:57
tags: redis
categories: redis
---

### Redis分布式锁
使用命令setnx占用锁，并设置一个锁超时时间，防止出现异常时能及时释放锁。
```shell
> setnx lock:codehole true
OK
> expire lock:codehole 5
... do something critical ...
> del lock:codehole
(integer) 1
```
但是这种方式万一在expire执行前出现了其他故障，如断电等就会造成死锁。
所以，在新版本中这样使用：（**推荐方式**）
```shell
> set lock:codehole true ex 5 nx
OK
... do something critical ...
> del lock:codehole
```

<!-- more -->