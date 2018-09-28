---
layout: post
title: redis进阶(1)
date: 2018-07-28 01:20:57
tags: redis
categories: redis
---

### 其它不常用命令

#### 位图数据
主要用于存放 bool 型数据，byte数组，使用位图操作 getbit/setbit 等将 byte 数组看成「位数组」来处理，大大减少节约存储空间。

>特别注意的是位操作存放的是ASCII码。

例子：存储一个“hello”，利用Python 命令行可以很方便地得到每个字符的 ASCII 码的二进制值。

```php
>>> bin(ord('h'))
'0b1101000'   # 高位 -> 低位
>>> bin(ord('e'))
'0b1100101'
>>> bin(ord('l'))
'0b1101100'
>>> bin(ord('l'))
'0b1101100'
>>> bin(ord('o'))
'0b1101111'
```
<!-- more -->

根据位数组的顺序和字符的位顺序是相反的，可以转换为位数组形式的显示方式（自动按照低位到高位进行扩展）。即只需要设置为1的位，所以h 字符只有 1/2/4 位需要设置，e 字符只有 9/10/13/15 位需要设置。
![](http://p2jr3pegk.bkt.clouddn.com/reids04-1.png)

所以可以这样操作redis存取（零存整取方式）：
```shell
>setbit s 1 1
(integer) 0
> setbit s 2 1
(integer) 0
> setbit s 4 1
(integer) 0
> setbit s 9 1
(integer) 0
> setbit s 10 1
(integer) 0
> setbit s 13 1
(integer) 0
> setbit s 15 1
(integer) 0
> get s
"he"
```
>「零存」就是使用 setbit 对位值进行逐个设置，「整存」就是使用字符串一次性填充所有位数组，覆盖掉旧值。

零存零取方式：
```shell
> setbit w 1 1
(integer) 0
> setbit w 2 1
(integer) 0
> setbit w 4 1
(integer) 0
> getbit w 1  # 获取某个具体位置的值 0/1
(integer) 1
> getbit w 2
(integer) 1
> getbit w 4
(integer) 1
> getbit w 5
(integer) 0
```

整存零取方式：
```shell
> set w h  # 整存
(integer) 0
> getbit w 1
(integer) 1
> getbit w 2
(integer) 1
> getbit w 4
(integer) 1
> getbit w 5
(integer) 0
```
如果对应位的字节是不可打印字符，redis-cli 会显示该字符的 16 进制形式：
```shell
> setbit x 0 1
(integer) 0
> setbit x 1 1
(integer) 0
> get x
"\xc0"
```

##### 位图统计指令 bitcount 和位图查找指令 bitpos
bitcount 用来统计指定位置范围内 1 的个数，bitpos 用来查找指定范围内出现的第一个 0 或 1。

```shell
> set w hello
OK
> bitcount w     #计算hello对应的位数组中所有1的个数
(integer) 21
> bitcount w 0 0  # 第一个字符中 1 的位数
(integer) 3
> bitcount w 0 1  # 前两个字符中 1 的位数
(integer) 7
> bitpos w 0  # 第一个 0 位
(integer) 0
> bitpos w 1  # 第一个 1 位
(integer) 1
> bitpos w 1 1 1  # 从第二个字符算起，第一个 1 位
(integer) 9
> bitpos w 1 2 2  # 从第三个字符算起，第一个 1 位
(integer) 17
```

##### 魔术指令 bitfield
一次可以操作多个位（最多可操作64位），bitfield 有三个子指令，分别是 get/set/incrby。
```shell
> set w hello
OK
> bitfield w get u4 0  # 从第一个位开始取 4 个位，结果是无符号数 (u)
(integer) 6
> bitfield w get u3 2  # 从第三个位开始取 3 个位，结果是无符号数 (u)
(integer) 5
> bitfield w get i4 0  # 从第一个位开始取 4 个位，结果是有符号数 (i)
1) (integer) 6
> bitfield w get i3 2  # 从第三个位开始取 3 个位，结果是有符号数 (i)
1) (integer) -3
```
简化的形式：
```shell
> bitfield w get u4 0 get u3 2 get i4 0 get i3 2
1) (integer) 6
2) (integer) 5
3) (integer) 6
4) (integer) -3
```
于是，我们可以利用set 子指令将第二个字符 e 改成 a，a 的 ASCII 码是 97：
```shell
> bitfield w set u8 8 97  # 从第 8 个位开始，将接下来的 8 个位用无符号数 97 替换
1) (integer) 101
> get w
"hallo"
```

在来看看incrby命令，特别注意溢出会将符号位丢掉（即折返）：

```shell
> set w hello
OK
> bitfield w incrby u4 2 1  # 从第三个位开始，对接下来的 4 位无符号数 +1
1) (integer) 11
> bitfield w incrby u4 2 1
1) (integer) 12
> bitfield w incrby u4 2 1
1) (integer) 13
> bitfield w incrby u4 2 1
1) (integer) 14
> bitfield w incrby u4 2 1
1) (integer) 15
> bitfield w incrby u4 2 1  # 溢出折返了
1) (integer) 0
```
>bitfield 指令提供了溢出策略子指令 overflow，用户可以选择溢出行为，默认是折返 (wrap)，还可以选择失败 (fail) 报错不执行，以及饱和截断 (sat)，超过了范围就停留在最大最小值。overflow 指令只影响接下来的第一条指令，这条指令执行完后溢出策略会变成默认值折返 (wrap)。...

```shell
> set w hello
OK
> bitfield w overflow sat incrby u4 2 1
1) (integer) 11
> bitfield w overflow sat incrby u4 2 2
1) (integer) 13
> bitfield w overflow sat incrby u4 2 2
1) (integer) 15
> bitfield w overflow sat incrby u4 2 1  # 保持最大值
1) (integer) 15
> bitfield w overflow fail incrby u4 2 1  # 不执行
1) (nil)
```

### HyperLogLog数据
三个指令 pfadd 、pfcount 和 pfmerge，根据字面意义很好理解，一个是增加计数，一个是获取计数，还有一个是用于将多个 pf 计数值累加在一起形成一个新的 pf 值。可用于计算网站的PV和UV等精度不是很高的统计需求。此命令功能用法类似set，但是统计百万级别数据更有优势，它需要占据一定 12k 的存储空间。
```shell
> pfadd codehole user1
(integer) 1
> pfcount codehole
(integer) 1
> pfadd codehole user2 user3 user4 user5
(integer) 1
> pfcount codehole
(integer) 5
```

#### 布隆过滤器

#### Redis-Cell限流

#### Geo存储结构

##### 存储地理位置信息
```shell
> geoadd company 116.48105 39.996794 juejin
(integer) 1
> geoadd company 116.514203 39.905409 ireader
(integer) 1
> geoadd company 116.489033 40.007669 meituan
(integer) 1
> geoadd company 116.562108 39.787602 jd 116.334255 40.027400 xiaomi
(integer) 2
```
##### 获取位置信息
```shell
> geopos company juejin
1) 1) "116.48104995489120483"
   2) "39.99679348858259686"
> geopos company ireader
1) 1) "116.5142020583152771"
   2) "39.90540918662494363"
> geopos company juejin ireader
1) 1) "116.48104995489120483"
   2) "39.99679348858259686"
2) 1) "116.5142020583152771"
   2) "39.90540918662494363"
```
##### 删除位置元素
geo 存储结构本质上是 zset，所以也可以用zset操作。
```shell
>zrem company xiaoxi
(integer) 1
```

##### 计算元素距离
geo甚至可以直接计算两者之间的距离，因为其内置了geohash距离算法。其中，最后的距离单位可以是 m、km、ml、ft，分别代表米、千米、英里和尺。
```shell
> geodist company juejin ireader km
"10.5501"
> geodist company juejin meituan km
"1.3878"
> geodist company juejin jd km
"24.2739"
> geodist company juejin xiaomi km
"12.9606"
> geodist company juejin juejin km
"0.0000"
```

##### 直接获取元素对应的 hash 值
geohash 可以获取元素的经纬度编码字符串，上面已经提到，它是 base32 编码。计算的值与[geohash.orghash/${hash}](http://geohash.org/)上应该是一致的。
```shell
> geohash company ireader
1) "wx4g52e1ce0"
> geohash company juejin
1) "wx4gd94yjn0"
```

##### 其他Geo相关特性---查询附近的元素和定位附近的元素
georadiusbymember 指令是最为关键的指令，它可以用来查询指定元素附近的其它元素，它的参数非常复杂。
```shell
# 范围 20 公里以内最多 3 个元素按距离正排，它不会排除自身
> georadiusbymember company ireader 20 km count 3 asc
1) "ireader"
2) "juejin"
3) "meituan"
# 范围 20 公里以内最多 3 个元素按距离倒排
> georadiusbymember company ireader 20 km count 3 desc
1) "jd"
2) "meituan"
3) "juejin"
# 三个可选参数 withcoord withdist withhash 用来携带附加参数
# withdist 很有用，它可以用来显示距离
> georadiusbymember company ireader 20 km withcoord withdist withhash count 3 asc
1) 1) "ireader"
   2) "0.0000"
   3) (integer) 4069886008361398
   4) 1) "116.5142020583152771"
      2) "39.90540918662494363"
2) 1) "juejin"
   2) "10.5501"
   3) (integer) 4069887154388167
   4) 1) "116.48104995489120483"
      2) "39.99679348858259686"
3) 1) "meituan"
   2) "11.5748"
   3) (integer) 4069887179083478
   4) 1) "116.48903220891952515"
      2) "40.00766997707732031"...
```

根据目标位置的经纬度，定位附近的元素：
```shell
> georadius company 116.514202 39.905409 20 km withdist count 3 asc
1) 1) "ireader"
   2) "0.0000"
2) 1) "juejin"
   2) "10.5501"
3) 1) "meituan"
   2) "11.5748"...
```
>对于Geo数据的redis尽量使用单独的Redis示例部署，不要使用集群环境

