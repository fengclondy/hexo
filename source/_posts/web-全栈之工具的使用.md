---
layout: post
title: web全栈-一些工具的使用
date: 2017-04-27 02:10:25
tags: web
categoriets: web
---

### 相关配置

#### eclipse默认编码为GBK，修改为UTF8的方法

都修改成UTF8的方法：

1、windows->Preferences...打开"首选项"对话框，左侧导航树，导航到general->Workspace，右侧 Text file encoding，选择Other，改变为UTF-8，以后新建立工程其属性对话框中的Text file encoding即为UTF-8。

2、windows->Preferences...打开"首选项"对话框，左侧导航树，导航到general->Content Types，右侧Context Types树，点开Text，选择Java Source File，在下面的Default encoding输入框中输入UTF-8，点Update，则设置Java文件编码为UTF-8。其他java应用开发相关的文件如：properties、XML等已经由Eclipse缺省指定，分别为ISO8859-1，UTF-8，如开发中确需改变编码格式则可以在此指定。


- 1.空间编码  ：Windows--Preferences-->General-->WorkSpace--(Text file encoding)
- 2.项目编码  ：右键工程Properties-->(Other）
- 3.文件编码  ：右键文件Properties-->(Other）

<!-- more -->

#### 安装lombok
1、下载lombok插件（https://projectlombok.org/download），将下载好的 lombok.jar 复制到 eclipse所在的文件夹目录下
2、②编辑eclipse.ini文件
在最后面插入以下两行：
```
-Xbootclasspath/a:lombok.jar
-javaagent:lombok.jar
```
3、重启eclipse

#### Navicate相关

- 1、Navicat可通过设置外键关联，当删除\更新某一主表的记录时，能够直接删除子表的记录，而不需要自己在去操作，可简化很多代码。

```
CASCADE
#在父表上update/delete记录时，同步update/delete掉子表的匹配记录 

SET NULL
#在父表上update/delete记录时，将子表上匹配记录的列设为null (要注意子表的外键列不能为not null)  

NO ACTION
#如果子表中有匹配的记录,则不允许对父表对应候选键进行update/delete操作  

RESTRICT
#同no action, 都是立即检查外键约束

SET NULL
#父表有变更时,子表将外键列设置成一个默认的值 但Innodb不能识别
```

#### 数据库相关

一、语法的不同(Sql和Oracle)
- 1、oracle没有`offet,limit`关键字，所以在oracle中要分页的话，要换成`rownum`。

- 2、oracle建表时，没有`auto_increment`，所有要想让表的主键自增，要自己添加序列。

- 3、oracle有一个`dual`表，当select后没有表时，加上的。不加会报错的。select 1 这个在mysql不会报错的，oracle下会。select 1 from dual这样的话，oracle就不会报错了。

- 4、对空值的判断，`name != ""`这样在mysql下不会报错的，但是oracle下会报错。在oracle下的要换成`name is not null`

- 5、oracle下对单引号，双引号是有要求的，一般不准用双引号，用了会报错（ERROR at line 1: ORA-00904: "t": invalid identifier），而mysql就没有这样的限制。

- 6、oracle表的字段是`number`型的，如果你$post得到的参数是123456，入库的时候，你需要用`to_number,to_date`这样的函数来强制转换一下，不然后会被当成字符串来处理。而mysql却不会。

- 7、group_concat这个函数，oracle是没有的，如果要想用自已写方法。

- 8、mysql的用户权限管理，是放到mysql自动带的一个数据库mysql里面的，而oracle是用户权限是根着表空间走的。（这点我感觉要好好理解下）

- 9、在oracle下用`group by`的话，group by后面的字段必须在`select`后面出现，不然会报错的，而mysql却不会。

- 10、mysql存储引擎有好多，常用的mysiam,innodb等，而创建oracle表的时候，好像只有一个存储引擎。

- 11、oracle字段无法选择位置，alter table add column before|after，这样会报错的，即使你用sql*plus这样的工具，也没法改字段的位置。

- 12、oracle的表字段类型也没有mysql多，并且有很多不同，例如：mysql的int,float对应oracle的number类型等。

- 13、oracle查询时from 表名后面 不能加上as不然会报错的，select t.username from test as t而在mysql下是可以的。

- 14、oracle中是没有substring这个函数的，mysql是有的。


二、mysql移植到oracle中的表类型转换

```
MYSQL                  ORACLE
BLOB（220）             RAW(220)
BLOB （20）             RAW(20)
BLOG(1024)             RAW(1024)
VARCHAR(n)             VARCHAR2(n)
CHAR                   CHAR
FLOAT(22,6)            NUMBER(22,6)
DOUBLE(44,12)          NUMBER(44,12)
TINYINT (3)            NUMBER(3)
SMALLINT(5)            NUMBER(5)
MEDIUMINT(8)           NUMBER(8)
INT(10)                NUMBER(10)
BIGINT(20)             NUMBER(20)
DATATIME               DATA
```


#### tomcat发布工程
http://www.cnblogs.com/george93/p/7478756.html


### Log4j的使用

#### 基本知识

Log4j有三个主要的组件：`Loggers`(记录器)，`Appenders` (输出源)和`Layouts`(布局)。这里可简单理解为日志类别，日志要输出的地方和日志以何种形式输出。　　

#### Log4j格式详解

`log4j.rootLogger = 日志级别，appender1, appender2, ….`

日志级别：`ALL<DEBUG<INFO<WARN<ERROR<FATAL<OFF`，不区分大小写

>注意，需在控制台输入，只需将其中一个appender定义为stdout即可。（只是定义输出源的名称）

>注意，rootLogger默认是对整个工程生效，如果只想对某些包操作，那么：log4j.logger.com.hutu=info, stdout，表示该日志对package com.hutu生效

`log4j.appender.appender1 = org.apache.log4j.日志输出到哪儿`


- `ConsoleAppender`（控制台），可选属性 Threshold （默认DEBUG）、ImmediateFlush（默认=true）、Target（默认=System.out）
- `FileAppender`（文件） ，可选属性 Threshold 、ImmediateFlush、Append（默认=true）、File
- `DailyRollingFileAppender`（每天产生一个日志文件），可选属性    Threshold 、ImmediateFlush、Append、DatePattern
- `RollingFileAppender`（文件大小到达指定尺寸时产生一个新的文件） ，可选属性 Threshold 、ImmediateFlush、Append、File、MaxFileSize、MaxBackupIndex
- `WriteAppender`（将日志信息以流格式发送到任意指定的地方）
- `JDBCAppender`（将日志信息保存到数据库中）


`log4j.appender.appender1.File=文件目录及文件${user.home}/logs/...`

`log4j.appender.appender1.MaxFileSize=最大文件大小`

`log4j.appender.appender1.MaxBackupIndex=备份文件个数`

`log4j.appender.appender1.DatePattern=对应格式文件（滚动日志）`

`log4j.appender.appender1.Threshold = 日志级别`

`log4j.appender.ServerDailyRollingFile.Append=true`

文件目录及文件，例如，/home/admin/logs/hutudan.log
最大文件大小，例如，100KB
备份文件个数，例如，1
- '.'yyyy-MM : 每月 ;  '.'yyyy-ww：每周 ;  '.'yyyy-MM-dd：每天 ;  '：每天两次
- '.'yyyy-MM-dd-HH：每小时 ;   '.'yyyy-MM-dd-HH-mm：每分钟
输出日记级别以上的 日志，例如：debug，info
追加上一次的文件后面，继续写

`log4j.appender.ServerDailyRollingFile.DatePattern=日志后缀格式`

例如，'.'yyyy-MM-dd
`log4j.appender.appender1.layout=org.apache.log4j.日志布局格式`

```
HTMLLayout（以HTML表格形式布局）
SimpleLayout（包含日志信息的级别和信息字符串）
TTCCLayout（包含日志产生的时间，执行绪，类别等信息）
PatternLayout（可以灵活的指定布局格式，常用）
```

`log4j.appender.appender1.layout.ConversionPattern=日志输出格式`

例如，%d - %m%n或%d{yyyy-MM-dd HH:mm:ss} %p [%c] %m%n
```
%c 输出日志信息所属的类的全名
%d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy-MM-dd HH:mm:ss }，输出类似：2002-10-18  22:10:28
%f 输出日志信息所属的类的类名
%l 输出日志事件的发生位置，即输出日志信息的语句处于它所在的类的第几行
%m 输出代码中指定的信息，如log(message)中的message
%n 输出一个回车换行符，Windows平台为“rn”，Unix平台为“n”
%p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL。如果是调用debug()输出的，则为DEBUG，依此类推
%r 输出自应用启动到输出该日志信息所耗费的毫秒数
%t 输出产生该日志事件的线程名
```

总结：
- Logger类：完成日志记录，设置日志信息级别
- Appender类：决定日志去向，终端、DB、硬盘
- Layout类：决定日志输出的样式，例如包含当前线程、行号、时间

附录：一张可供参考的配置:

```
#定义LOG输出级别
log4j.rootLogger=INFO,Console,F,E

#定义mybatis日志输出
log4j.logger.org.mybatis=INFO,ibatis
log4j.logger.com.software.dao=INFO,ibatis
log4j.additivity.org.mybatis=false
log4j.appender.ibatis=org.apache.log4j.DailyRollingFileAppender
log4j.appender.ibatis.File=../logs/shuichan/ibatis/ibatis.log
log4j.appender.ibatis.layout=org.apache.log4j.PatternLayout
log4j.appender.ibatis.layout.ConversionPattern=[%-5p] %d{yyyy-MM-dd HH:mm:ss} method:%c%n%m%n

#定义日志输出目的地为控制台
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.Target=System.out
log4j.appender.Console.layout = org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%c%n%m%n

#可以灵活地指定日志输出格式，下面一行是指定具体的格式
#log4j.appender.Console.layout = org.apache.log4j.PatternLayout
#log4j.appender.Console.layout.ConversionPattern=[%c] - %m%n

#文件大小到达指定尺寸的时候产生一个新的文件
log4j.appender.F = org.apache.log4j.RollingFileAppender
#指定输出目录
log4j.appender.F.File = ../logs/shuichan/logfile.log
log4j.appender.F.Append = true
#定义文件最大大小
log4j.appender.F.MaxFileSize = 10MB
# 输出所以日志，如果换成DEBUG表示输出DEBUG以上级别日志
log4j.appender.F.Threshold = INFO
log4j.appender.F.layout = org.apache.log4j.PatternLayout
log4j.appender.F.layout.ConversionPattern =[%p] [%d{yyyy-MM-dd HH\:mm\:ss,SSS}][%c]%m%n

### 输出ERROR 级别以上的日志到=E://logs/error.log ###
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File = ../logs/shuichan/error.log 
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR 
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss,SSS}  [ %t:%r ] - [ %p ]  %c%m%n
```

