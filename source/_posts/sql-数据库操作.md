---
layout: post
title: sql数据库操作
date: 2018-05-02 05:35:01
tags: sql
categoriets: sql
---

### 基本操作

1、获取所有可用的数据库：`SHOW DATABASES`；

2、选择数据库(customers)：`USE customers`；

3、用于显示数据库服务器的状态信息：`SHOW STATUS`；

4、用来显示授权用户的安全权限：`SHOW GRANTS`；

5、用来显示数据库服务器或警告信息：`SHOW ERRORS` 或者 `SHOW WARNINGS`；

6、用于显示创建数据库时的创建语句：`SHOW CREATE DATABASE customers`；

7、用于显示创建表时的创建语句：`SHOW CREATE TABLE customers`；

8、获取当前所选的数据库中所有可用的表：`SHOW TABLES`；

9、获取表中所有列的信息：`SHOW COLUMNS FROM tableName`；同时DESCRIBE语句有相同的效果：`DESCRIBE tableName`；

更多基本操作参考：https://juejin.im/post/5ae55861f265da0ba062ec71

### 语法详解

1、MySQL 和 Oracle 中的限制返回大小

Mysql语法：
```
SELECT column_name(s)
FROM table_name
LIMIT number;
```

Oracle语法：

```  
SELECT column_name(s)
FROM table_name
WHERE ROWNUM <= number;
```

2、Sql中的连接：

- `INNER JOIN`：如果表中有至少一个匹配，则返回行
- `LEFT JOIN`：即使右表中没有匹配，也从左表返回所有的行
- `RIGHT JOIN`：即使左表中没有匹配，也从右表返回所有的行
- `FULL JOIN`：只要其中一个表中存在匹配，则返回行

3、`UNION` 操作符用于合并两个或多个 `SELECT` 语句的结果集（无重复的值）。`UNION ALL`也是（所有值，不包含重复的）

4、`INSERT INTO SELECT` 语句从一个表复制数据，然后把数据插入到一个已存在的表中。目标表中任何已存在的行都不会受影响。

5、Sql约束：

- `NOT NULL` - 指示某列不能存储 NULL 值。
- `UNIQUE` - 保证某列的每行必须有唯一的值。
- `PRIMARY KEY` - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。
- `FOREIGN KEY` - 保证一个表中的数据匹配另一个表中的值的参照完整性。
- `CHECK` - 保证列中的值符合指定的条件。
- `DEFAULT` - 规定没有给列赋值时的默认值。

6、`Auto-increment` 会在新记录插入表中时生成一个唯一的数字，通常用于主键约束。

7、Sql函数：

```
AVG() - 返回平均值
COUNT() - 返回行数
FIRST() - 返回第一个记录的值
LAST() - 返回最后一个记录的值
MAX() - 返回最大值
MIN() - 返回最小值
SUM() - 返回总和
UCASE() - 将某个字段转换为大写
LCASE() - 将某个字段转换为小写
MID() - 从某个文本字段提取字符
LEN() - 返回某个文本字段的长度
ROUND() - 对某个数值字段进行指定小数位数的四舍五入
NOW() - 返回当前的系统日期和时间
FORMAT() - 格式化某个字段的显示方式      FORMAT(column_name,format) 
```

