---
layout: post
title: SSM框架开发 笔记
date: 2017-06-27 02:53:43
tags: 
---

### SSM框架开发 笔记

1、前台ajax到后台，后台要返回给前台。数据都正常，但前台就是获取不到值。---解决办法：`@ResponseBody`写在Controller上，这样前台才能获取到该值。

2、关于post和ajax：

`$.post`和`$.get`是`$.ajax`的两种简单形式。如果对业务要求不是很好，可以用，否则应该使用 ajax。因为 ajax 可以指定很多其他参数，比如 post 默认是异步执行的，而 ajax 可以直接指定异步、同步方式。

3、SSM框架，mybatis配置xml，需要在插入一条数据之后返回插入的id，只需在 {这里} insert into XX的头部插入如下语句即可。

```xml
<selectKey resultType="java.lang.Integer" order="AFTER" keyProperty="id">
     SELECT LAST_INSERT_ID() AS id
</selectKey>
```

<!-- more -->

4、SSM框架的 foreach 批量操作

```xml
<foreach collection="list" item="item" index="index" separator="," >  
        (#{item.userid},#{item.typename},#{item.productcom},#{item.unitprice},
        #{item.unit},#{item.feedtype},#{item.createtime})  
</foreach>
```

foreach属性的参数说明:
>- item： 必选参数。循环体中的形参，具体说明：在list和数组中是其中的对象，在map中是value。
- collection ： 必选参数。循环提中的实参，说明 `List<?>`对象默认用list代替作为键，数组对象有array代替作为键，Map对象用map代替作为键（默认情况）。当然，也可以使@Param("keyName")来设置键。如果入参是User对象， User有属性List ids，那么这个collection = "ids"，含义是对哪个元素做循环。
- separator：可选参数。元素之间的分割符。
- open：可选参数。与close 参数连用。含义：foreach代码的开始和结束符号，例如：open="("  close=")"。
- index：可选参数。在List中是元素的序号，在Map中是元素的key。

下面给出几个例子参考：

>insert into FixBuy (UserId, TypeName, ProductCom, UnitPrice, Unit, FeedType,CreateTime) values  (?, ?) , (?, ?)，(?, ?) , (?, ?)

```xml
<insert id="insertBatch"  parameterType="java.util.List">  
  insert into FixBuy (UserId, TypeName, ProductCom, UnitPrice, Unit, FeedType,CreateTime)
  values
  <foreach collection="list" item="item" index="" separator="," >  
    (#{item.userid},#{item.typename},#{item.productcom},#{item.unitprice},
    #{item.unit},#{item.feedtype},#{item.createtime})  
  </foreach>  
</insert>  
//说明: 从上一层传来的函数入参是 List<FixBuy>
```

>select count(*) from users WHERE id in ( ? , ? ,?) ；

```xml
<select id="selectCountById" resultType="java.lang.String" parameterType="java.util.List">    
select count(*) from users    
  <where>    
    id in    
    <foreach item="item" collection="list" separator="," open="(" close=")" index="">    
      #{item.id, jdbcType=NUMERIC}    
    </foreach>    
  </where>    
</select>   
```

>补充：一定要注意到$和#的区别，$的参数直接输出，#的参数会被替换为?，然后传入参数值执行。（还不是很明白其中道理）

5、使用 update 更新时，传入参数是实体类 pojo，总报错 java.lang.NullPointerException

一定要注意实体类的 get 方法，对于值为 null 的直接返回即可（因为 SSM 会对实体类的每个属性进行一次get方法），不能操作。尤其是日期格式的问题，需特别注意

6、使用 if  test 判断的时候，一定不能加在 where 语句后面，where 语句不能识别，会直接将其作为入参进行判断。

```xml
<if test="#{0} != null">
      username=#{0} And
</if>
```

检错的方法：log4j.properties里面配置DEBUG，进行显示。
```properties
log4j.rootLogger=DEBUG,Console
log4j.logger.org.mybatis=DEBUG
```

7、SQL语法：

- `isnull(expr)`的用法：如expr 为 null，那么 isnull() 的返回值为 1，否则返回值为 0。

- `IFNULL(expr1,expr2)`的用法：假如expr1 不为 NULL，则 IFNULL() 的返回值为 expr1; 否则其返回值为expr2。IFNULL() 的返回值是数字或是字符串，具体情况取决于其所使用的语境。

- `NULLIF(expr1,expr2)`的用法：如果 expr1=expr2 成立，那么返回值为 NULL，否则返回值为 expr1。