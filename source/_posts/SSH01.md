---
layout: post
title: SSH框架开发 笔记
date: 2018-05-02 06:15:31
tags:
---

### Spring学习

#### 一、原理

1、spring的核心机制是IOC(反转控制、依赖注入)和AOP(面向切面编程)。

注1：spring是一站式的框架,对EE的三层有每一层的解决方案,Web层,业务层,数据访问层.Web层:SpringMVC , 持久层:JDBC Template , 业务层 : Spring的Bean管理。 IOC思想 : 工厂+反射+配置文件,底层原理就是提供一个工厂Bean,然后提供一个配置文件,把一些类全都配置在配置文件中,通过xml解析获得类的全路径,从而反射获得类的实例。

注2：AOP就是面向切面编程,通过预编译的方式和运行期动态代理实现程序功能的统一维护的技术。主要的功能是 : 日志记录,性能统计,安全控制,事务处理,异常处理等。

2、概念：

　　　(1)当摸个角色需要另一个角色协助的时候，传统的程序设计过程中，通常由调用者来创建被调用者的实例。但在Spring中穿件被调用者的实例不在由被调用者来完成，因此称为控制反转。创建被调用者的工作由Spring来完成，然后注入调用者，因此也称为依赖注入。（特征：降低了组建的耦合性）

        (2)面向对象编程将程序分解成各个层次的对象，面向切面编程将程序运行过程分解成各个切面。（特征：各步骤之间良好的隔离型，源代码无关性）

 

#### 二、事务

1、事务

事务:是逻辑上的一组操作,要么全部成功,要么全部失败

事务的特性 :   ACID

- Atomicity 原子性: 事务不可分隔
- Consister 一致性: 事务执行的前后,数据完整性保持一致.
- Isolation 隔离性: 一个事务执行的时候,不应该收到其他事务的干扰
- Durability 持久性: 一旦结束,数据永久的保存到数据库

2、spring中的事务管理

 
分层开发: 事务处于service层

Spring的事务管理分成两类:

编程式事务管理:    手动编写代码完成事务管理.
声明式事务管理:    不需要手动编写代码,配置
spring中的注解：
```
@Component("")
@Service("")   --装配Bean,标示为service类
@Repository("") --装配Bean,标示为dao类
@Controller("") --装配Bean,标示为controller类
@Autowired @Qualifier("userDao"): --在类中注入Bean
@Aspect --定义切面
@Before("execution(* com.luogg.demo1.UserDao.*(..))") ---前置增强
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml") --单元测试注解
@Transactional --声明事务
```

>这里讲下声明事务管理的操作：配置XML文件,（包括事务管理,连接池的配置）  --->   在需要使用注解的方法前边加上注解@Transactional

```
<!-- 引入外部属性文件. -->
<context:property-placeholder location="classpath:jdbc.properties"/>
     
   <!-- 配置c3p0连接池 -->
   <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
         <property name="driverClass" value="${jdbc.driver}"/>
         <property name="jdbcUrl" value="${jdbc.url}"/>
         <property name="user" value="${jdbc.user}"/>
         <property name="password" value="${jdbc.password}"/>
   </bean>
     
   <!-- 业务层类 -->
   <bean id="accountService" class="cn.itcast.spring3.demo4.AccountServiceImpl">
         <!-- 在业务层注入Dao -->
         <property name="accountDao" ref="accountDao"/>
   </bean>
     
   <!-- 持久层类 -->
   <bean id="accountDao" class="cn.itcast.spring3.demo4.AccountDaoImpl">
        <!-- 注入连接池的对象,通过连接池对象创建模板. -->
        <property name="dataSource" ref="dataSource"/>
   </bean>
    
   <!-- 事务管理器 -->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
         <property name="dataSource" ref="dataSource"/>
   </bean>
   
<!-- 开启注解的事务管理 -->
<tx:annotation-driven/>
```



### hibernate中多表映射关系配置


#### 1.one-to-many一对多关系的映射配置(在一的一方实体映射文件中配置)

```
<!-- 
            cascade属性:级联操作属性
                save-update: 级联保存,保存客户时,级联保存客户关联的联系人
                delete:级联删除,删除客户时,级联删除客户关联的联系人
                all:级联保存+级联删除
-->
<!-- 
            inverse属性:设置是否不维护关联关系
                true:不维护关联
                false(默认值):维护关联
 -->
        <!-- 一对多 -->
        <set name="linkMen" inverse="true" >
            <!-- 外键列名 -->
            <key column="lkm_cust_id" ></key>
            <!-- 该集合是一对多关系表达,关联的对象时linkman -->
            <one-to-many class="LinkMan" />
        </set>   
```

一对多|多对一关系中,放置sql语句冗余.一般选择一的一方放弃维护,inverse属性设置为true.

 
#### 2.many-to-one 多对一关系映射配置(在多的一方实体映射文件中配置)

```
<!-- 
            cascade属性:级联操作属性
                save-update: 级联保存,保存客户时,级联保存客户关联的联系人
                delete:级联删除,删除客户时,级联删除客户关联的联系人
                all:级联保存+级联删除
 -->
 <!-- 
             没有inverse属性:
                 外键列所在实体,无法放弃维护关联关系.
 -->
        <!-- 多对一 -->
        <many-to-one name="customer"    
         column="lkm_cust_id" 
         class="Customer" ></many-to-one>
```
 

#### 3.many-to-many多对多关系映射配置

```
<!-- 多对多关系配置 
                table:中间表表名
-->
<!-- 
            inverse属性:设置是否不维护关联关系
                true:不维护关联
                false(默认值):维护关联
-->
<!-- 
            cascade属性:级联操作属性
                save-update: 级联保存,保存客户时,级联保存客户关联的联系人
                delete:级联删除,删除客户时,级联删除客户关联的联系人
                all: 级联保存+级联删除
 -->
        <set name="roles" table="sys_user_role"   >
            <!-- 别人引用"我"的外键列名 -->
            <key column="user_id" ></key>
            <!-- 表达集合是多对多关系
                class属性:表达我与谁是多对多
                column属性:表达另外一个外键列名
             -->
            <many-to-many class="Role" column="role_id" ></many-to-many>
        </set>
```
多对多关系中,选择一方发起维护关系,放置中间表数据录入重复,根据业务逻辑决定,如商品和订单是多对多关系,订单维护商品放弃维护

