---
layout: post
title: redis入门（3）
date: 2018-07-17 07:26:42
tags: redis
categories: redis
---

### redis其他的一些命令

#### scan
迭代方式，不是很懂。据说可以使用`scan 0`代替`keys`

#### bigkeys
在启动服务的时候，`redis-cli -p 6380 --bigkeys`: 查看当前缓存的一个总体状况

#### slowlog
```shell
SLOWLOG subcommand [argument]
```
subcommand主要有：
- get，用法：slowlog get [argument]，获取argument参数指定数量的慢日志。
- len，用法：slowlog len，总慢日志数量。
- reset，用法：slowlog reset，清空慢日志。

<!-- more -->

#### monitor
开启监控。监控redis服务器上的所有操作

#### info
查看redis的运行状态参数。

#### config
redis运维常用参数。`config get *`和`config set XX XXX `

### redis其他设置
在config配置文件里，配置。可以避免测试环境中的命令影响到生产环境
```shell
rename-command flushdb flushddbb
rename-command flushall flushallall
rename-command keys keysys
```

