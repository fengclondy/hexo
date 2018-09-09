---
layout: post
title: springDataJpa学习(上)
date: 2018-02-09 04:06:09
tags: springDataJpa
categories: springDataJpa
---

>orm框架的本质是简化编程中操作数据库的编码，发展到现在基本上就剩两家了，
一个是宣称可以不用写一句SQL的hibernate，一个是可以灵活调试动态sql的mybatis.  

>hibernate特点就是所有的sql都用Java代码来生成，不用跳出程序去写（看）sql，有着编程的完整性，
发展到最顶端就是spring data jpa这种模式了

### 基本操作实现（默认方法）

1、继承JpaRepository

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

2、使用默认方法

```
@Test
public void testBaseQuery() throws Exception {
	User user=new User();
	userRepository.findAll();
	userRepository.findOne(1l);
	userRepository.save(user);
	userRepository.delete(user);
	userRepository.count();
	userRepository.exists(1l);
	// ...
}
```

### Repository中查询方法定义规则（自定义方法）

| Keyword | Sample | JPQL snippet |
|---------|--------|--------------|
| And | findByLastNameAndFirstname   | ...where x.lastName = ?1 and x.firstName = ?2 |
| Or | findByLastNameOrFirstname  | ...where x.lastName = ?1 or x.firstName = ?2 |
| Is,Equals | findByFirstnameIs,findByFirstnameEquals | ...where x.firstname = ?1 |
| Between | findByStartDateBetween   | ...where x.startDate  between 1? and ?2 |
| LessThan | findByAgeLessThan   | ...where x.age < ?1 |
| LessThanEqual	| findByAgeLessThanEqual | ...where x.age <= ?1 |
| GreateThan | findByAgeGreaterThan  | ...where x.age > ?1 |
| GreaterThanEqual | findByAgeGreaterThanEqual | ...where x.age >= ?1 |
| After | findByStartDateAfter  | ...where x.startDate > ?1 |
| Before | findByStartDateBefore   | ...where x.startDate < ?1 |
| IsNull | findByAgeIsNull   | ...where x.age is null |
| IsNotNull,NotNull | findByAge(is)NotNul   | ...where x.age not null |
| Like | findByFirstNameLike   | ...where x.firstName like ?1 |
| NotLike | findByFirstNameNotLike   | ...where x.firstName not like ?1 |
| StartingWith | findByFirstNameStartingWith   | ...where x.firstName like ?1(parameter bound with appended %) |
| EndingWith | findByFirstNameEndingWith   | ...where x.firstName like ?1(parameter bound with prepended %) |
| Containing | findByFirstNameContaining   | ...where x.firstName like ?1(parameter bound wrapped in %) |
| OrderBy | findByAgeOrderByLastNameDesc   | ...where x.age =?1 order by x.lastName desc |
| Not | findByLastNameNot   | ...where x.lastName <>?1 |
| In | findByAgeIn(Collection<Age> ages)   | ...where x.age in ?1 |
| NotIn | findByAgeNotIn(Collection<Age> ages)  | ...where x.age not in ?1 |
| TRUE | findByActiveTrue()   | ...where x.active = true |
| FALSE | findByActiveFalse()   | ...where x.active = false |
| IgnoreCase | findByFirstnameIgnoreCase | ...where UPPER(x.firstame) = UPPER(?1) |


### 实例

1、简单查询

```java
//where name like ?% and age <?
public List<User> findByNameStartingWithAndAgeLessThan(String name,Integer age);
//where name like %? and age >?
public List<User> findByNameEndingWithAndAgeLessThan(String name,Integer age);
//where name in(?,?,?) or age <?
public List<User> findByNameInOrAgeLessThan(List<String> names,Integer age);
//where name in(?,?,?) and age <?
public List<User> findByNameInAndAgeLessThan(List<String> names,Integer age);
```

2、分页等复杂查询  

在查询的方法中，需要传入参数`Pageable` ,当查询中有多个参数的时候`Pageable`建议做为最后一个参数传入

```java
Page<User> findALL(Pageable pageable);
    
Page<User> findByUserName(String userName,Pageable pageable);
```

使用：

```java
@Test
public void testPageQuery() throws Exception {
	int page=1,size=10;  //页数、每页条数
	Sort sort = new Sort(Direction.DESC, "id");  //排序规则
    Pageable pageable = new PageRequest(page, size, sort);
    userRepository.findALL(pageable);
    userRepository.findByUserName("testName", pageable);
}
```

对于排序，除了方法名的`OrderBy`外，还可以传入`Sort`参数：

`Page<User> findByUserName(String userName,Sort sort);`

使用：

`userRepository.findByUserName("testName", new Sort(Direction.DESC, "id"));`


3、限制查询 

只需要查询前N个元素，或者支取前一个实体。例如：

```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

>>此处特别注意下，findBy和findXXXBy是一样的，Jpa返回的都是整个实体类。如果需要返回某一列属性，需要借助`@Query`实现.


### 一些注解的含义

#### 主键生成策略

通过annotation来映射hibernate实体的,基于annotation的hibernate主键标识为`@Id`, 
其生成规则由`@GeneratedValue`设定的.这里的`@Id`和`@GeneratedValue`都是JPA的标准用法.

GenerationType有四种方式：
- TABLE：使用一个特定的数据库表格来保存主键。
- SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。（ORACLE数据库）
- IDENTITY：主键由数据库自动生成（主要是自动增长型）。（Mysql数据库）
- AUTO：主键由程序控制。（默认方式）

自增实例:

```
### ORACLE中使用
@Id  
@GeneratedValue(strategy =GenerationType.SEQUENCE,generator="aaa")  
@SequenceGenerator(name="aaa", sequenceName="seq_payment", allocationSize=1)
```

在org.hibernate.id.IdentifierGeneratorFactory中指定了hibernate主键生成策略和各自的具体生成器之间的关系：

```
static {  
  GENERATORS.put("uuid", UUIDHexGenerator.class);  
  GENERATORS.put("hilo", TableHiLoGenerator.class);  
  GENERATORS.put("assigned", Assigned.class);  
  GENERATORS.put("identity", IdentityGenerator.class);  
  GENERATORS.put("select", SelectGenerator.class);  
  GENERATORS.put("sequence", SequenceGenerator.class);  
  GENERATORS.put("seqhilo", SequenceHiLoGenerator.class);  
  GENERATORS.put("increment", IncrementGenerator.class);  
  GENERATORS.put("foreign", ForeignGenerator.class);  
  GENERATORS.put("guid", GUIDGenerator.class);  
  GENERATORS.put("uuid.hex", UUIDHexGenerator.class); //uuid.hex is deprecated  
  GENERATORS.put("sequence-identity", SequenceIdentityGenerator.class);  
}  
```

示例（使用uuid作为主键生成策略）：

```
  @Id
  @GeneratedValue(generator = "IDGenerator")
  @GenericGenerator(name = "IDGenerator", strategy = "uuid")
```

下面两种形式是等价的：

```
//方式1
@Id
@GeneratedValue(generator = "IDGenerator")
@GenericGenerator(name = "IDGenerator", strategy = "identity", allocationSize=1)

//方式2
@Id
@GeneratedValue(strategy=GenerationType.IDENTITY)
```


### 弊端

>1、方法名过长，约定大于配置  
 2、难实现特别复杂的查询
 
spring data完美支持，使用`@Query`和`@Modifying`来进行改善：  
`@Query`支持原生Sql语句，标注在方法头部即可。`@Modifying`主要用于更新操作。
可添加`@Transactional`对事物的支持，查询超时的设置等


#### 实例

```java
@Query("select u from User u where u.name=?1 and u.age=?2")
public List<User> queryParams1(String name,String age);  //方式1

@Query("select u from User u where u.name=:name and u.age=:age")
public List<User> queryParams2(@Param("name") String name, (@Param("age") String age); //方式2

@Modifying
@Query("update User u set u.userName = ?1 where u.id = ?2")
int modifyByIdAndUserId(String  userName, Long id);
	
@Transactional
@Modifying
@Query("delete from User where id = ?1")
void deleteByUserId(Long id);

@Transactional(timeout = 10)
@Query("select u from User u where u.emailAddress = ?1")
User findByEmailAddress(String emailAddress);
    
### 复杂一点的查询
public static final String QUERY_SQL = "select id from (select * from TargetInfo when user_id=?1) where type!=?2 order by id desc";

@Query(value = RepositoryQuery.QUERY_SQL, nativeQuery = true)
List<BigInteger> findAllByTypeAndGroupByUserId(long userId, int type);

```

#### 其他

PagingAndSortingRespsitory ：分页显示，jpa的子类