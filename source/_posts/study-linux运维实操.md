---
layout: post
title: study--linux系统运维
date: 2018-07-20 07:18:15
tags: study
categories: study
---
### top命令查看系统使用情况

使用系统命令top即可看到如下类似信息：
```
top - 15:14:11 up 17 days,  1:31,  4 users,  load average: 3.89, 12.82, 12.27
Tasks: 213 total,   1 running, 207 sleeping,   0 stopped,   5 zombie
%Cpu(s):  2.2 us,  1.5 sy,  0.0 ni, 96.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3788432 total,   138816 free,  3479284 used,   170332 buff/cache
KiB Swap:  2097148 total,   476760 free,  1620388 used.    62780 avail Mem 
```

第三行Cpu参数详解：

>us 用户空间占用CPU百分比；sy 内核空间占用CPU百分比；ni 用户进程空间内改变过优先级的进程占用CPU百分比；
id 空闲CPU百分比；wa 等待输入输出的CPU时间百分比；hi 硬件中断；si 软件中断；st: 实时；


### free查看内存使用情况

`free -m`查看的是（MB）的内存状态
```
              total        used        free      shared  buff/cache   available
Mem:           3699        2648         856          31         195         797
Swap:          2047        1706         341
```
>total：内存总数；used：已经使用的内存数；free：空闲的内存数；shared：当前已经废弃不用；buffers Buffer：缓存内存数；cached Page：缓存内存数。

free总的可用内存数: (指的第一部分Mem行中的free + buffers + cached)

