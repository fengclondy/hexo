---
layout: post
title: springBoot --- 用@Scheduled实现定时任务
date: 2018-06-01 07:16:05
tags: springBoot
categories: springBoot
---


### 定时任务的实现

```
## 方式一、固定时间
@Scheduled(fixedDelay = 5000)  //上次执行完成后，每 5s 调用一次
public void doSomething() { 
    // something that should execute periodically
}


## 方式二、固定速率（周期）
@Scheduled(fixedRate = 5000)    //上次执行时间点后，每 5s 调用一次
public void doSomething() { 
    // something that should execute periodically
}

## 方式三、
@Scheduled(cron = "0 0 2 * * ?")　　//每天凌晨两点执行
public void doSomething(){
    // something that should execute periodically
}

>注：如果执行某次操作时，方法被阻塞了，下次方法到来时并不会立马执行，都会先去执行上次没有执行的任务。
```

- cron方式参数说明：

```
按顺序依次为:
1)秒（0~59）
2)分钟（0~59）
3)小时（0~23）
4)天（月）（0~31）
5)月（0~11 或 1-12 或者 JAN-DEC）
6)天（星期）（1-7 或者 SUN-SAT）
7)年份（1970－2099）（该项不一定存在，一般前六个即可）

>注："*"字符代表所有可能的值;"/"字符用来指定数值的增量("0/15"表示从第0分钟开始，每15分钟);
"？"字符仅被用于天（月）和天（星期）两个子表达式，表示不指定值(如果2个子表达式之一被指定，需将另一个置为"?");
"L"字符仅被用于天（月）和天（星期）两个子表达式，它是单词"last"的缩写("6L"表示这个月的倒数第６天，"ＦＲＩＬ"表示这个月的最一个星期五);
"-"字符用来指定一个时间段。
```
>可以包含列表：
>>天（星期））可以为 “MON-FRI”，“MON，WED，FRI”，“MON-WED,SAT”