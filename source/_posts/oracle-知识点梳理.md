---
layout: post
title: oracle知识点梳理
date: 2018-01-15 11:18:49
tags: oracle
categories: oracle
---

### 时间处理

oracle中的时间处理和mysql中的不太一样

- 时间格式转string用：`to_char(sysdate,'yyyy-mm-dd hh24:mi:ss')`

- string转时间格式用：`to_date()`

有关时间处理的函数也有一些,列举常用的：

##### extract函数
 ```sql
 select extract(year from sysdate) as year from dual; //获取系统时间的当前年  
 select extract(month from sysdate) from dual;  
 select extract(day from sysdate) from dual;  
 ```
##### trunc函数
```sql
 select trunc(sysdate,'yyyy') from dual;  
 //获取（系统时间）当前年的一月一号
```
##### add_months函数
 ```sql
 select add_months(sysdate,-12) from dual //获取一年前的当前时间
 ```

<!-- more -->

###### test测试

 ```sql
 select extract(year from sysdate)-1||'-01-01 00:00:00' start_time,concat(extract(year from sysdate)-1||'-12-'
 ||to_char(LAST_DAY(sysdate),'dd '),'23:59:59') end_time   from dual;
 ```
##### 联合函数union与union all
 ```
Union：对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；

Union All：对两个结果集进行并集操作，包括重复行，不进行排序；

Intersect：对两个结果集进行交集操作，不包括重复行，同时进行默认规则的排序；

Minus：对两个结果集进行差操作，不包括重复行，同时进行默认规则的排序
 ```

##### 自增长主键

 需要自己创建创建序列和触发器。 创建触发器时要留意触发器的触发条件


##### bug修复
 1、`ORA-00979  GROUP BY 表达式错误`

 原因： select 列表项中不存在的列可以出现在group by的列表项中，但反过来就不行了，
 在select列表项中出现的列必须全部出现在group by后面(聚合函数除外)

##### navicat操作数据库的一些体会

 1. oracle表和字段是有大小写的区别。oracle默认是大写，如果我们用双引号括起来的就区分大小写，
 如果没有，系统会自动转成大写。查询的时候体会不出来，但是对具体字段进行AS操作的时候，一定要加""。

 2. oracle对数据类型要求很严格，尤其是对日期格式的处理，经常出现问题的地方也是在这里。


 #### 语法参考

 ```
 -------------------------------多表查询--------------------------------------
 
 
--笛卡尔积查询.
select * from emp,dept;--结果为64条,emp表有14条,dept表有4条
--别名查询
--改变查询后的结果显示的列名,在字段后面写要显示的列名(注意一定要用双引号括起来,oracle查询中之后这里用到双引号!)
select ename "name" ,loc "地点" from emp e ,dept d where e.deptno = d.deptno;
 
--查询出雇员的编号,姓名,部门的编号和名称,地址
select * from emp;
select * from dept;
select e.empno  "雇员编号",
       e.ename  "姓名",
       e.deptno "部门的编号",
       d.dname  "部门名称",
       d.loc    "部门地址"from emp e,
       dept   d where e.deptno = d.deptno;--当字段名唯一时,字段前的别名可以省略
select empno  "雇员编号",
      ename  "姓名",
      e.deptno "部门的编号",--当两张表中都有该字段时,字段前的别名一定不能省略
      dname  "部门名称",
      loc    "部门地址"from emp e,
      dept     d where e.deptno = d.deptno;
 
--查询出每个员工的上级领导
select e1.mgr,e1.ename , e2.empno, e2.ename
  from emp e1, emp e2
 where e1.mgr = e2.empno ;
 
--在上面的查询基础上查询该员工的部门名称
select e1.mgr,e1.ename , e2.empno, e2.ename,d.dname
  from emp e1, emp e2,dept d
 where e1.mgr = e2.empno and e1.deptno = d.deptno;
 
--查出每个员工编号,姓名,部门名称,工资等级和他的上级领导的姓名,工资等级
select * from salgrade;
select * from emp;
 
select e1.mgr,
       e1.sal,
       e1.ename,
       decode(g1.grade,
              1,
              '一等',
              2,
              '二等',
              3,
              '三等',
              4,
              '四等',
              5,
              '五等') "员工工资等级",
       e2.empno,
       e2.sal,
       e2.ename,
       decode(g2.grade,
              1,
              '一等',
              2,
              '二等',
              3,
              '三等',
              4,
              '四等',
              5,
              '五等') "领导工资等级"
  from emp      e1, --员工表
       emp      e2, --领导表
       salgrade g1, --员工工资等级
       salgrade g2 --领导工资等级
 where e1.mgr = e2.empno
   and e1.sal between g1.losal and g1.hisal
   and e2.sal between g2.losal and g2.hisal;
 
----------------------------外连接(左右连接)---------------------------
--查询所有部门下的员工
 
select * from emp;
select * from dept;
select e.ename, e.deptno, d.dname
  from emp e,--非全量表,因为在emp表中没有40对应的部门编号
  dept d--全量表 dept表中有所有的部门编号
 where e.deptno(+) = d.deptno;--(+)是oracle特有的表示左连接或者有链接的符号,
                               --(+)添加在非全量表中(那个表中少,就添加到那个表中)
select e.ename, e.deptno, d.dname
  from emp e
 right join dept d
    on e.deptno = d.deptno; --右连接实现
select e.ename, e.deptno, d.dname
  from dept d
  left join emp e
    on e.deptno = d.deptno; --左连接实现
--查询出比雇员7654的工资高,同时从事和7788的工作一样的员工
select *
  from emp e
 where sal > (select sal from emp where empno = 7654)
   and job = (select job from emp where empno = 7788);
<br>--------------------------子查询--------------------------<br>
--查询出每个部门的最低工资和最低工资的雇员和部门名称
select ename, sal, e.deptno,d.dname
  from emp e,
   (select deptno, min(sal) minsal from emp group by deptno) a,
   dept d
 where e.sal = a.minsal and e.deptno = d.deptno;



---------------------分页功能--------------------
rowid:--是每行数据指向磁盘的物理地址
rownum:--查询时每行数据上贴的序号
select e.*, rownum,rowid from emp e ;
--查询员工表中工资最高的前三名
select t.*, rownum
  from (select e.* from emp e order by sal desc) t
 where rownum < 4;
--查询4,5,6的工资排名的员工
select t2.*
  from (select t.*, rownum r
          from (select e.* from emp e order by sal desc) t
         where rownum < 7) t2
 where r > 3;
  
 select t2.*
  from (select t.*, rownum r
          from (select e.* from emp e order by sal desc) t
         ) t2
 where r > 3 and r < 7;
  
 --------------------------行转列------------------------------
 --统计每年入职的员工个数
 select to_char(hiredate,'yyyy'), count(*) from emp group by to_char(hiredate,'yyyy');
  
 decode(列名,值,显示数据) --
 别名
  
 select sum(counts) "Total",
         sum(decode(years,'1980',counts)) "1980",
         sum(decode(years,'1982',counts)) "1982",
         sum(decode(years,'1981',counts)) "1981",
         sum(decode(years,'1987',counts)) "1987"
   from
   (select to_char(hiredate,'yyyy') years, count(*) counts from emp group by to_char(hiredate, 'yyyy'));
---子查询的总结
   --子查询可以返回单行单列值
   --子查询可以返回多行多列值
   --子查询可以返回多行多列值(当成子表使用)
   --子查询可以提高查询效率
  
 ------------------exists的用法------------
 select * from 表名 where exists (子查询)
 exists(子查询) 如果子查询没有结果,exists(子查询)返回false
 exists(子查询) 如果子查询有结果,exists(子查询)返回true
 --查询没有员工的部门
 select * from dept d where not exists(select * from emp e where e.deptno = d.deptno )
  
 -----------------------------集合运算---------------------------------
 并集
 交集
 补集
 --查询工资大于1500或者是20号部门下的员工
 select * from emp where sal > 1500
 union -- 并集
 select * from emp where deptno = 20 ;
 select * from emp where sal > 1500 or deptno = 20;
 --查询工资大于1500并且是20号部门下的员工
 select * from emp where sal > 1500
 intersect -- 交集
 select * from emp where deptno = 20 ;
 select * from emp where sal > 1500 and deptno = 20;
 --查询1981年入职的员工,不包括经理和总裁
 select * from emp where to_char(hiredate,'yyyy')='1981'
 minus -- 补集
 select * from emp where job='MANAGER' or job = 'PRESIDENT'
 --如果两个查询结果对应的列数和数据类型都一样,就可以用集合运算
  
 ----------------递归查询(oracle独有)---------------------
 select * from 表名
 start with 条件1
 connect by prior 条件2
 --查询7566号员工的所有下属
 select * from emp
 start with empno=7566
 connect by prior empno=mgr
 ```

 