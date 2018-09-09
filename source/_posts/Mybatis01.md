---
layout: post
title: Mybatis01
date: 2018-05-17 02:33:25
tags: mybatis
categories: mybatis
---


#### #{}和${}
`#{}`是预编译处理，`${}`是字符串替换。

Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值,该方式可有效的防止SQL注入，提高系统安全性；

Mybatis在处理${}时，就是把${}替换成变量的值。

#### 实体类中的属性名和表中的字段名不一样
方法1：通过Sql查询语句的别名设置返回字段名
方法2：通过xml文件的`<resultMap>`字段来完成映射关系，自动转换。

<!-- more -->

#### 模糊查询like语句
方法1：在Java代码中添加sql通配符。形如：
```
string wildcardname = “%smi%”; 
list<name> names = mapper.selectlike(wildcardname);

<select id=”selectlike”> 
    select * from foo where bar like #{value} 
</select>
```

方法2：在SQL中拼接通配符：
```
string wildcardname = “smi”; 
   list<name> names = mapper.selectlike(wildcardname);

   <select id=”selectlike”> 
    select * from foo where bar like "%"#{value}"%"
</select>
```

#### 插入数据返回id

有一些数据库如Oracle，并不支持`AUTO_INCREMENT`列属性，它使用序列（SEQUENCE）来产生主键的值。

我们可以使用`@SelectKey`注解来为任意SQL语句来指定主键值，作为主键列的值。

假设我们有一个名为 STUD_ID_SEQ 的序列来生成 STUD_ID 主键值。

```sql
@Insert("INSERT INTO STUDENTS(STUD_ID,NAME,EMAIL,ADDR_ID, PHONE) VALUES (#{studId},#{name},#{email},#{address.addrId},#{phone})")  
@SelectKey(statement="SELECT STUD_ID_SEQ.NEXTVAL FROM DUAL", keyProperty="studId", resultType=int.class, before=true)  
int insertStudent(Student student);  
```

这里我们使用了@SelectKey来生成主键值，并且存储到了student对象的studId属性上。由于我们设置了before=true,该语句将会在执行INSERT语句之前执行。

如果你使用**序列**作为触发器来设置主键值，我们可以在INSERT语句执行后，从`sequence_name.currval`获取数据库产生的主键值。

```sql
@Insert("INSERT INTO STUDENTS(NAME,EMAIL,ADDR_ID, PHONE) VALUES (#{name},#{email},#{address.addrId},#{phone})")  
@SelectKey(statement="SELECT STUD_ID_SEQ.CURRVAL FROM DUAL", keyProperty="studId", resultType=int.class, before=false)  
int insertStudent(Student student);  
```

https://blog.csdn.net/u013214151/article/details/52211614