---
layout: post
title: MMM--数据库方面
date: 2018-08-04 01:09:51
tags:
---


##### 编写一个 SQL 查询来实现分数排名。如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。
>嵌套子查询
```
+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
例如，根据上述给定的 Scores 表，你的查询应该返回（按分数从高到低排列）：
+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```
```
### 方式1
select Score,
(select count(distinct Score) from Scores as s2 where s2.Score >= s1.Score) Rank 
from Scores as s1
order by Score DESC;

### 方式2
select s1.Score, count(distinct s2.Score) Rank
from Scores as s1 join Scores as s2 on s1.Score <= s2.Score
group by s1.Id 
order by s1.Score DESC;
```

<!-- more -->

##### SQL查询应该返回 员工工资表 中第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null

>可直接使用IFNULL()这个方法。
```
+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
返回结果应为：
+----------+
| Employee |
+----------+
| Joe      |
+----------+
```

```
### 方法一
select IFNULL((select Distinct Salary from Employee order by Salary DESC limit 1,1),null) as SecondHighestSalary;

### 方法二
select max(Salary) as SecondHighestSalary from Employee where Salary<(select max(Salary) from Employee);
```


##### 给定一个Weather表，编写一个SQL查询来查找与之前(昨天的)日期相比温度更高的所有日期的id。
>运用一些内置函数辅助完成。
```
+---------+------------------+------------------+
| Id(INT) | RecordDate(DATE) | Temperature(INT) |
+---------+------------------+------------------+
|       1 |       2015-01-01 |               10 |
|       2 |       2015-01-02 |               25 |
|       3 |       2015-01-03 |               20 |
|       4 |       2015-01-04 |               30 |
+---------+------------------+------------------+
例如，根据上述给定的 Weather 表格，返回如下 Id:
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```

```
### 方法一
select w1.Id from weather w1
inner join weather w2 on w1.Temperature > w2.Temperature and DATEDIFF(w1.RecordDate, w2.RecordDate) = 1;

### 方法二
SELECT w1.Id FROM Weather w1, Weather w2
WHERE w1.Temperature > w2.Temperature AND TO_DAYS(w1.RecordDate)=TO_DAYS(w2.RecordDate) + 1;

### 方法三
SELECT w1.Id FROM Weather w1, Weather w2
WHERE w1.Temperature > w2.Temperature AND SUBDATE(w1.RecordDate, 1) = w2.RecordDate;
```

##### 编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。
>group by可直接对邮箱进行过滤
```
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
根据以上输入，你的查询应返回以下结果：

+---------+
| Email   |
+---------+
| a@b.com |
+---------+
说明：所有电子邮箱都是小写字母。
```

```
Select Email From Person Group By Email Having Count(*) > 1
```

##### 删除 Person 表中重复的电子邮件
>建立临时表。
```
delete p1 from Person p1,Person p2 where p1.Email = p2.Email and p1.Id>p2.Id;
```


##### 有一个courses 表 ，有: student (学生) 和 class (课程)。请列出所有超过或等于5名学生的课。
>先分组，然后再过滤。
```
+---------+------------+
| student | class      |
+---------+------------+
| A       | Math       |
| B       | English    |
| C       | Math       |
| D       | Biology    |
| E       | Math       |
| F       | Computer   |
| G       | Math       |
| H       | Math       |
| I       | Math       |
+---------+------------+
应该输出:

+---------+
| class   |
+---------+
| Math    |
+---------+
```

```
SELECT class from courses group by class having count(DISTINCT student) >= 5; 
```

##### 找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。
>取奇数：`id&1`和匹配 `not like'%boring%'`，注意没有is关键字
>`%（匹配任意次数的任意字符）`,`_(只匹配一个字符)`,`*(匹配前面的子表达式零次或多次)`。注意和java中的是有区别的
```
例如，下表 cinema:

+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
对于上面的例子，则正确的输出是为：

+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
```

```
select * from cinema where description not like'boring' and id&1 order by rating
```

##### 给定一个 salary表，如下所示，有m=男性 和 f=女性的值 。交换所有的 f 和 m 值(例如，将所有 f 值更改为 m，反之亦然)。要求使用一个更新查询，并且没有中间临时表。
>使用if函数。
```
例如:

| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
运行你所编写的查询语句之后，将会得到以下表:

| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
```

```
update salary set sex = if(sex='m','f','m')
```