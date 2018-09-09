---
layout: post
title: redis入门（2）
date: 2018-08-01 07:25:42
tags: redis
categories: redis
---


### 数据结构基础教程

#### String类型
>字符串最大长度为512M。 

初始化字符串：
```shell
> set ireader beijing.zhangyue.keji.gufen.youxian.gongsi
OK
```
获取字符串内容：
```shell
> get ireader
"beijing.zhangyue.keji.gufen.youxian.gongsi"
```
获取字符串长度：
```shell
> strlen ireader
(integer) 42
```
获取子串：
```shell
> getrange ireader 28 34
"youxian"
```
<!-- more -->

覆盖子串：

```shell
> setrange ireader 28 wooxian
(integer) 42  # 返回长度
> get ireader
"beijing.zhangyue.keji.gufen.wooxian.gongsi"
```
追加子串：
```shell
> append ireader .hao
(integer) 46 # 返回长度
> get ireader
"beijing.zhangyue.keji.gufen.wooxian.gongsi.hao"
```
计数器（前提：传入的字符串是一个整数）
```shell
> set ireader 42
OK
> get ireader
"42"
> incrby ireader 100
(integer) 142
> get ireader
"142"
> decrby ireader 100
(integer) 42
> get ireader
"42"
> incr ireader  # 等价于incrby ireader 1
(integer) 143
> decr ireader  # 等价于decrby ireader 1
(integer) 142
```
>计数器是有范围的，它不能超过Long.Max(即，9223372036854775807)，不能低于Long.MIN(即，-9223372036854775808).
强行操作，会报错`(error) ERR increment or decrement would overflow`

过期和删除。字符串可以使用del指令进行主动删除，可以使用expire指令设置过期时间，到点会自动删除，这属于被动删除。可以使用ttl指令获取字符串的寿命。
```shell
> expire ireader 60
(integer) 1  # 1表示设置成功，0表示变量ireader不存在
> ttl ireader
(integer) 50  # 还有50秒的寿命，返回-2表示变量不存在，-1表示没有设置过期时间
> del ireader
(integer) 1  # 删除成功返回1
> get ireader
(nil)  # 变量ireader没有了
```

组合使用：
```shell
> setex name 5 codehole  # 5s 后过期，等价于 set+expire

> setnx name codehole  # 如果 name 不存在就执行 set 创建,否则不执行创建工作
```


#### List类型（双向列表，非数组）

队列／堆栈。链表可以从表头和表尾追加和移除元素，结合使用rpush/rpop/lpush/lpop四条指令，
可以将链表作为队列或堆栈使用，左向右向进行都可以。
可以使用正负下标，负标表示倒数，0和-n都表示第一个元素。
>在日常应用中，列表常用来作为异步队列来使用。
```shell
# 右进左出
> rpush ireader go
(integer) 1
> rpush ireader java python
(integer) 3
> lpop ireader
"go"
> lpop ireader
"java"
> lpop ireader
"python"
# 左进右出
> lpush ireader go java python
(integer) 3
> rpop ireader
"go"
...
# 右进右出
> rpush ireader go java python
(integer) 3
> rpop ireader 
"python"
...
# 左进左出
> lpush ireader go java python
(integer) 3
> lpop ireader
"python"
...
```
获取链表长度:
```shell
> rpush ireader go java python
(integer) 3
> llen ireader
(integer) 3
```
随机读：
```shell
> rpush ireader go java python
(integer) 3
> lindex ireader 1
"java"
> lrange ireader 0 2
1) "go"
2) "java"
3) "python"
> lrange ireader 0 -1  # -1表示倒数第一
1) "go"
2) "java"
3) "python"
```
修改元素:
```shell
> rpush ireader go java python
(integer) 3
> lset ireader 1 javascript
OK
> lrange ireader 0 -1
1) "go"
2) "javascript"
3) "python"
```
插入元素:(通过方向参数before/after来指明是前置或后置插入，指定的是具体位置的值，而非坐标)
```shell
> rpush ireader go java python
(integer) 3
> linsert ireader before java ruby
(integer) 4
> lrange ireader 0 -1
1) "go"
2) "ruby"
3) "java"
4) "python"
```
删除元素：(指定删除的最大个数以及元素的值)
```shell
> rpush ireader go java python
(integer) 3
> lrem ireader 1 java
(integer) 1
> lrange ireader 0 -1
1) "go"
2) "java"
```
定长列表：(两个参数start和end，范围之内的元素被保留，范围之外的都将被移除；如果end小于start，相当于del命令，全部删除)
```shell
> rpush ireader go java python javascript ruby erlang rust cpp
(integer) 8
> ltrim ireader -3 -1
OK
> lrange ireader 0 -1
1) "erlang"
2) "rust"
3) "cpp"
```

#### Hash类型

增加元素：
```shell
> hset ireader go fast
(integer) 1
> hmset ireader java fast python slow
OK
```
获取元素：
```shell
> hmset ireader go fast java fast python slow
OK
> hget ireader go
"fast"
> hmget ireader go python
1) "fast"
2) "slow"
> hgetall ireader
1) "go"
2) "fast"
3) "java"
4) "fast"
5) "python"
6) "slow"
> hkeys ireader
1) "go"
2) "java"
3) "python"
> hvals ireader
1) "fast"
2) "fast"
3) "slow"
```
删除元素：
```shell
> hmset ireader go fast java fast python slow
OK
> hdel ireader go
(integer) 1
> hdel ireader java python
(integer) 2
```
判断元素是否存在：
```shell
> hmset ireader go fast java fast python slow
OK
> hexists ireader go
(integer) 1
```
计数器（前提：传入的字符串是一个整数）
```shell
> hincrby ireader go 1
(integer) 1
> hincrby ireader python 4
(integer) 4
> hincrby ireader java 4
(integer) 4
> hgetall ireader
1) "go"
2) "1"
3) "python"
4) "4"
5) "java"
6) "4"
> hset ireader rust good
(integer) 1
> hincrby ireader rust 1
(error) ERR hash value is not an integer
```
扩容和缩容。Java的HashMap只有扩容，Redis采用了渐进式rehash的方案。它会同时保留两个新旧hash结构，避免单线程造成的卡顿现象。

#### Set类型（HashSet）
增加元素：
```shell
> sadd ireader go java python
(integer) 3
```
读取元素：
```shell
> sadd ireader go java python
(integer) 3
> smembers ireader
1) "java"
2) "python"
3) "go"
> scard ireader
(integer) 3
> srandmember ireader
"java"
```
删除元素：
```shell
> sadd ireader go java python rust erlang
(integer) 5
> srem ireader go java
(integer) 2
> spop ireader    #随机删除的
"erlang"
```
判断元素是否存在：(只能传单个值)
```shell
> sadd ireader go java python rust erlang
(integer) 5
> sismember ireader rust
(integer) 1
> sismember ireader javascript
(integer) 0
```

#### ZSet类型
SortedSet(zset)是Redis提供的一个非常特别的数据结构，
一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，
另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

增加元素：（value/score对，score放在前面）
```shell
> zadd ireader 4.0 python
(integer) 1
> zadd ireader 4.0 java 1.0 go
(integer) 2
```
长度：
```shell
> zcard ireader
(integer) 3
```
删除元素：
```shell
> zrem ireader go python
(integer) 2
```
计数器（前提：传入的字符串是一个整数）
```shell
> zadd ireader 4.0 python 4.0 java 1.0 go
(integer) 3
> zincrby ireader 1.0 python
"5"
```
获取排名和分数：
```shell
> zscore ireader python
"5"
> zrank ireader go  #正向排序；分数低的排名靠前，rank值小
(integer) 0
> zrank ireader java
(integer) 1
> zrank ireader python
(integer) 2
> zrevrank ireader python   #反向排序；分数高的排名靠前
(integer) 0
```
根据排名范围获取元素列表:（withscores参数显示获取元素的权重）
```shell
> zrange ireader 0 -1  # 获取所有元素
1) "go"
2) "java"
3) "python"
> zrange ireader 0 -1 withscores
1) "go"
2) "1"
3) "java"
4) "4"
5) "python"
6) "5"
> zrevrange ireader 0 -1 withscores
1) "python"
2) "5"
3) "java"
4) "4"
5) "go"
6) "1"
```
根据score范围获取列表并排序：(参数-inf表示负无穷，+inf表示正无穷)
```shell
> zrangebyscore ireader 0 5
1) "go"
2) "java"
3) "python"
> zrangebyscore ireader -inf +inf withscores  #正向排序
1) "go"
2) "1"
3) "java"
4) "4"
5) "python"
6) "5"
> zrevrangebyscore ireader +inf -inf withscores  #反向排序，注意正负反过来了
1) "python"
2) "5"
3) "java"
4) "4"
5) "go"
6) "1"
```
根据范围移除元素列表:
```shell
> zremrangebyrank ireader 0 1
(integer) 2  # 删掉了2个元素
> zadd ireader 4.0 java 1.0 go
(integer) 2
> zremrangebyscore ireader -inf 4
(integer) 2
> zrange ireader 0 -1
1) "python"
```
跳跃列表。zset内部的排序功能是通过「跳跃列表」数据结构来实现的，它的结构非常特殊，也比较复杂。






