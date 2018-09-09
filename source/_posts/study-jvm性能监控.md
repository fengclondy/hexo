---
layout: post
title: study-jvm性能监控
date: 2018-08-14 01:55:55
tags: study
categories: study
---


### JVM性能监控

参考：https://blog.csdn.net/maosijunzi/article/details/46049117
     https://blog.csdn.net/he90227/article/details/52136154

依据
- GC日志
- 堆转储快照（heapdump/hprof文件）
- 线程快照（threaddump/javacore文件）
- 运行日志
- 异常堆栈

分析依据的工具
- jps：显示指定系统内的所有JVM进程，如：`jps -l`
- jstat：收集JVM各方面的运行数据，如: `jstat -guctil PID`
- jinfo：显示JVM配置信息，格式：`jinfo -flag parameter PID`
- jmap：形成堆转储快照（heapdump文件），格式： `jmap -dump:format=b,file=文件名 PID`；查看最占内存元素：`jmap -histo PID`
- jhat：分析heapdump文件，格式：``jhat 文件名`
- jstack：显示JVM的线程快照(定位线程长时间卡顿的原因（线程间死锁、死循环、请求外部资源导致的长时间等待），格式：`jstack -l PID`
- jconsole    #图形化界面
- visualVM   #图形化界面

 <!-- more -->

例子：

```
C:\Users\george>jstat -gcutil 8588
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
 71.04   0.00  19.35  30.91  98.02  96.82     18    0.178     2    0.211    0.389
```
>说明：S0(from区)使用了71.04%；S1(to区)使用了0；E(Eden区)使用了19.35%；O(Old，年老代)使用了30.91%；M(Metadata，元数据区使用比例)使用了98.02%；
YGC(Young GC)了18次，YGCT(Young GC Time)花销0.178秒；FGC(Full GC)了2次，FGCT(Full GC Time)花销0.211秒；GCT(GC Time)总花销0.389秒。

```
C:\Users\george>jstat -class 8588
Loaded  Bytes  Unloaded  Bytes     Time
  9605 17329.8        0     0.0      13.33
```
说明：加载了9605个类，总共占有17329.8字节；卸载了0个类，卸载的类的字节数为0，类的加载与卸载共花销13.33秒

### JVM调优
jvm调优主要是优化内存,体现在：(1)各个代的大小;(2)垃圾收集器选择.

1、调整代大小的常见参数：
```
-Xmx
-Xms
-Xmn
-XX:SurvivorRatio
-XX:MaxTenuringThreshold
-XX:PermSize
-XX:MaxPermSize
```

2、垃圾收集的选择
- Parallel Scavenge/Parallel Old
- ParNew/CMS (首选)


