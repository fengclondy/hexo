---
layout: post
title: spring--事务管理
date: 2017-04-27 02:32:56
tags: spring
categoriets: spring
---

### Spring事务管理

#### 一、基础知识

- 1、Spring中的事务分为物理事务和逻辑事务

物理事务：就是底层数据库提供的事务支持，如JDBC或JTA提供的事务；

逻辑事务：是Spring管理的事务，不同于物理事务，逻辑事务提供更丰富的控制，而且如果想得到Spring事务管理的好处，必须使用逻辑事务，因此在Spring中如果没特别强调一般就是逻辑事务；

- 2、@Transactional可指定的事务属性
```
@isolation：用于指定事务的隔离级别。默认为底层事务的隔离级别

@noRollbackFor：指定遇到特定异常时不回滚事务

@noRollbackForClassName：指定遇到特定的多个异常时不回滚事务

@propagation：指定事务传播行为

@readOnly：指定事务是否可读

@rollbackFor：指定遇到特定异常时回滚事务

@rollbackForClassName：指定遇到特定的多个异常时回滚事务

@timeout：指定事务的超长时长。
```

- 3、属性参数的含义

事务隔离级别：用来解决并发事务时出现的问题，其使用TransactionDefinition中的静态变量来指定：

```
ISOLATION_DEFAULT：默认隔离级别，即使用底层数据库默认的隔离级别；
ISOLATION_READ_UNCOMMITTED：未提交读；
ISOLATION_READ_COMMITTED：提交读，一般情况下我们使用这个；
ISOLATION_REPEATABLE_READ：可重复读；
ISOLATION_SERIALIZABLE：序列化。
```

事务传播行为：Spring管理的事务是逻辑事务，而且物理事务和逻辑事务最大差别就在于事务传播行为，事务传播行为用于指定在多个事务方法间调用时，事务是如何在这些方法间传播的，Spring共支持7种传播行为：

```
PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价PROPAGATION_REQUIRED。
```

#### 二、注解@Transactional用法

>这里以之前遇到的一个项目做简要说明

- 1、在配置文件的头部引入<tx>和<aop>命名空间（以spring-mybatis为例）

```
 <beans
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:tx="http://www.springframework.org/schema/tx"
xsi:schemaLocation="                                          
 http://www.springframework.org/schema/tx                       
 http://www.springframework.org/schema/tx/spring-tx-4.0.xsd                   
 http://www.springframework.org/schema/aop        
 http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

//同时，加入事务配置并开启该事务:
<bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="dataSource" ref="dataSource" />
</bean>
//记得一定要开启
<tx:annotation-driven proxy-target-class="false" transaction-manager="transactionManager"/>
```

- 2、在需要进行事务处理的方法或者类头上加上

`@Transactional(propagation = Propagation.REQUIRES_NEW,isolation = Isolation.DEFAULT,rollbackFor = Exception.class)`

>注：被申明为事务的方法一定要抛异常，且该异常不能被try{}cathch{}捕获

 
#### 三、注意要点

- 一个类的内部，方法 A 调用方法 B 。在方法 B 上申明事务而 A 没有，则方法 B 的事务不起作用。如果在 A 上申明而 B 上没有，则方法 B 会自动参与该事务。（重要）

- 在类上申明@Tranactional，相当于在所有的方法上申明@Tranactional。如果不涉及多表的关联操作问题，一般不需要声明。

- `try{}catch{}`和`rollbackFor`是矛盾的（个人理解）。要想实现事务的回滚，必须有 Exception 产生，如果 Exception 被捕获，那么注解`@Tranactional`申明就不起作用。
  从被调用处到调用处，需要向上抛Exception,直到其被上层捕获。

对于业务逻辑复杂的可根据注解+事务编程来实现。

编程式事务的实现：

```
Connection conn = null;  
UserTransaction tx = null;  
try {  
    tx = getUserTransaction();                       //1.获取事务  
    tx.begin();                                    //2.开启JTA事务  
    conn = getDataSource().getConnection();           //3.获取JDBC  
    //4.声明SQL  
    String sql = "select * from INFORMATION_SCHEMA.SYSTEM_TABLES";  
    PreparedStatement pstmt = conn.prepareStatement(sql);//5.预编译SQL  
    ResultSet rs = pstmt.executeQuery();               //6.执行SQL  
    process(rs);                                   //7.处理结果集  
    closeResultSet(rs);                             //8.释放结果集  
    tx.commit();                                  //7.提交事务  
} catch (Exception e) {  
    tx.rollback();                                 //8.回滚事务  
    throw e;  
} finally {  
   conn.close();                                //关闭连接  
}  
```