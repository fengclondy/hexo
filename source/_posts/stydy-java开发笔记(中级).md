---
layout: post
title: java开发笔记(中级)
date: 2018-08-06 07:48:23
tags: study
categories: study
---

#### Navicat可通过设置外键关联，当删除,更新某一主表的记录时，不需要自己在去操作，简化代码

```
CASCADE
在父表上update/delete记录时，同步update/delete掉子表的匹配记录 

SET NULL
在父表上update/delete记录时，将子表上匹配记录的列设为null (要注意子表的外键列不能为not null)  

NO ACTION
如果子表中有匹配的记录,则不允许对父表对应候选键进行update/delete操作  

RESTRICT
同no action, 都是立即检查外键约束

SET NULL
父表有变更时,子表将外键列设置成一个默认的值 但Innodb不能识别
```

#### 实体类中排除非表中的字段
1.使用 transient 修饰
`private transient String noColumn;`

2.使用 static 修饰
`private static String noColumn;`

3.使用 TableField 注解
```
@TableField(exist=false)
private String noColumn;
```

#### 事物机制
数据库事务中的四大特性，ACID。即：A:原子性(Atomicity)，C:一致性(Consistency)，I:隔离性(Isolation)，D:持久性(Durability)。

```
###事物传播行为
@Transactional(propagation=Propagation.REQUIRED) ：如果有事务, 那么加入事务, 没有的话新建一个(默认情况下)
@Transactional(propagation=Propagation.NOT_SUPPORTED) ：容器不为这个方法开启事务
@Transactional(propagation=Propagation.REQUIRES_NEW) ：不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务
@Transactional(propagation=Propagation.MANDATORY) ：必须在一个已有的事务中执行,否则抛出异常
@Transactional(propagation=Propagation.NEVER) ：必须在一个没有的事务中执行,否则抛出异常(与Propagation.MANDATORY相反)
@Transactional(propagation=Propagation.SUPPORTS) ：如果其他bean调用这个方法,在其他bean中声明事务,那就用事务.如果其他bean没有声明事务,那就不用事务.
###事物超时设置
@Transactional(timeout=30) //默认值为-1表示永不超时
###事物隔离级别
@Transactional(isolation = Isolation.READ_UNCOMMITTED)：读取未提交数据(会出现脏读, 不可重复读) 基本不使用
@Transactional(isolation = Isolation.READ_COMMITTED)：读取已提交数据(会出现不可重复读和幻读) （SqlServer默认的级别）
@Transactional(isolation = Isolation.REPEATABLE_READ)：可重复读(会出现幻读) （Mysql默认的级别）
@Transactional(isolation = Isolation.SERIALIZABLE)：串行化
```
- 1、脏读 : 一个事务读取到另一事务未提交的更新数据  
- 2、不可重复读 : 在同一事务中, 多次读取同一数据返回的结果有所不同, 换句话说, 
后续读取可以读到另一事务已提交的更新数据. 相反, "可重复读"在同一事务中多次
读取数据时, 能够保证所读数据一样, 也就是后续读取不能读到另一事务已提交的更新数据  
- 3、幻读 : 一个事务读到另一个事务已提交的insert数据  

>注意点:
1、@Transactional 只能被应用到public方法上, 对于其它非public的方法,如果标记了@Transactional也不会报错,但方法没有事务功能.

>2、用 spring 事务管理器,由spring来负责数据库的打开,提交,回滚.默认遇到运行期例外(throw new RuntimeException("注释");)会回滚，即遇到不受检查（unchecked）的例外，即Error时回滚；
而遇到需要捕获(checked)的例外(throw new Exception("注释");)不会回滚,即遇到受检查的例外（就是非运行时抛出的异常，编译器会检查到的异常叫受检查例外或说受检查异常）时，
需我们指定方式来让事务回滚要想所有异常都回滚,要加上 @Transactional( rollbackFor={Exception.class,其它异常}) .如果让unchecked例外不回滚： @Transactional(notRollbackFor=RunTimeException.class)

>3、@Transactional 注解应该只被应用到 public 可见度的方法上。 如果你在 protected、private 或者 package-visible 的方法上使用 @Transactional 注解，它也不会报错，
 但是这个被注解的方法将不会展示已配置的事务设置。

>4、@Transactional 注解可以被应用于接口定义和接口方法、类定义和类的 public 方法上。然而，请注意仅仅 @Transactional 注解的出现不足于开启事务行为，它仅仅是一种元数据，
能够被可以识别 @Transactional 注解和上述的配置适当的具有事务行为的beans所使用。上面的例子中，其实正是元素的出现开启了事务行为。
xml文件需要配置：
```
<beans:bean id="transactionManager"  
    class="org.springframework.orm.jpa.JpaTransactionManager">  
    <beans:property name="dataSource" ref="dataSource" />  
    <beans:property name="entityManagerFactory" ref="entityManagerFactory" />  
</beans:bean>
  
<!-- 声明使用注解式事务 -->  
<tx:annotation-driven transaction-manager="transactionManager" /> 
```

>5、在具体的类（或类的方法）上使用 @Transactional 注解，而不要使用在类所要实现的任何接口上。你当然可以在接口上使用 @Transactional 注解，
但是这将只能当你设置了基于接口的代理时它才生效。因为注解是不能继承的，这就意味着如果你正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装（将被确认为严重的）。

>6、不同类之间的方法调用，如类A的方法a()调用类B的方法b()，只要方法a()或b()配置了事务，运行中就会开启事务，产生代理。若两个方法都配置了事务，两个事务具体以何种方式传播，取决于设置的事务传播特性

>7、同一类内方法调用，假定类A的方法a()调用方法b()，若a()没有配置事物，即使被调用的b()方法配置了事务，此事务在被调用时也不生效。如在a()上申明了事物，而b()没有，则调用b()的时候会自动加入到a的事物中。


##### 分布式事物机制
CAP定理，又被叫作布鲁尔定理，即一致性，高可用和分区容错性。实际情况下，分布式系统理论上不可能保证100%可靠，所以一般只能选择CP或者AP架构。

对于CP来说，放弃可用性，追求一致性和分区容错性，我们的zookeeper其实就是追求的强一致。

对于AP来说，放弃一致性(这里说的一致性是强一致性)，追求分区容错性和可用性，这是很多分布式系统设计时的选择，常见的的有BASE理论。

[参考文档](https://blog.csdn.net/congyihao/article/details/70195154)
解决策略：
1.两阶段提交(Two Phase Commit, 2PC), 具有强一致性, 是CP系统的一种典型实现.
2.TCC, 是基于补偿型事务的AP系统的一种实现, 具有最终一致性.
3.异步确保型，通过将一系列同步的事务操作变为基于消息执行的异步操作, 避免了分布式事务中的同步阻塞操作的影响.
4.最大努力通知型，与前面异步确保型操作不同的一点是, 在消息由MQ Server投递到消费者之后, 允许在达到最大重试次数之后正常结束事务.

>如果业务场景需要强一致性, 那么尽量避免将它们放在不同服务中, 也就是尽量使用本地事务, 避免使用强一致性的分布式事务.
如果业务场景能够接受最终一致性, 那么最好是使用基于消息的最终一致性的方案(异步确保型)来解决.
如果业务场景需要强一致性, 并且只能够进行分布式服务部署, 那么最好是使用TCC方案而不是2PC方案来解决.


#### linux下批量注释代码
批量注释：ctrl+V然后上下键选中操作行 -->按下大写I-->输入"#"或者"\\"-->按两下ESC，即可。


#### @autowired报空指针
`@autowired`写在变量上的注入要等到类完全加载完，才会将相应的bean注入，而变量是在加载类的时候按照相应顺序加载的，所以变量的加载要早于`@autowired`变量的加载


取消注释：
