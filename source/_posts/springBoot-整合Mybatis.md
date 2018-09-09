---
layout: post
title: springBoot和Mybatis整合
date: 2018-01-17 14:00:04
tags: springBoot
categories: springBoot
---

>有两种解决方案:一种是使用注解解决一切问题；一种是简化后的老传统。

### 无配置文件注解版

1.pom.xml 配置文件中添加：

```xml
<dependency>
   <groupId>org.mybatis.spring.boot</groupId> 
   <artifactId>mybatis-spring-boot-starter</artifactId><!--支持mybatis相关-->
   <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId> <!--数据库Mysql-->
    <scope>runtime</scope>
</dependency>
```

2、application.properties 添加相关配置

```properties
mybatis.type-aliases-package=com.neo.entity

spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.url = jdbc:mysql://localhost:3306/test1?useUnicode=true&characterEncoding=utf-8
spring.datasource.username = root
spring.datasource.password = root
```

<!-- more -->

3、启动类加入包扫描`@MapperScan`：

```java
@SpringBootApplication
@MapperScan("com.neo.mapper")
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

4.创建Mapper接口，使用注解配置SQL和方法之间的关系，接口如下

```java
public interface UserMapper {
    @Select( "select * from user" )
    @Results( {
              @Result( property = "uid", column = "id" ),
              @Result( property = "username", column = "name" )
          } )
      /**查找全部用户信息*/
    public List<User> findAll();
    
    @Insert( "insert into user values (null,#{username},#{age})" )
    
      /**添加用户信息(并返回插入的id)*/
    public void inser( User user );
    
    @Update("update user set name=#{userName} WHERE id =#{uid}")
	  /**更新用户信息*/
    void update( User user );
    
    @Delete( "delete from user where id = #{uid}" )
      /**根据 id 删除用户信息*/
    public void delete( Integer uid );
}
```
>插入的时候返回插入的主键id(三种方式)：

`@SelectKey(statement="select @@IDENTITY", keyProperty="id", before=false, resultType=int.class)`
`@SelectKey(statement="sselect LAST_INSERT_ID()", keyProperty="id", before=false, resultType=int.class)`
`@SelectKey(statement="select max(id) id from table", keyProperty="id", before=false, resultType=int.class)`

当然，也可以用另外一种方法：

```
@Options(useGeneratedKeys = true,keyProperty = "实体类的id字段",keyColumn = "数据库主键字段")
```

稍微复杂一点的可能需要这样写：

**批量操作**，以**插入**为例：

- 方法一：

```
    /** 批量插入消息*/
    @InsertProvider(type = UserDaoMapper.class, method = "saveAll")
    void saveAll(@Param("user") Collection<User> user);
```

UserDaoMapper(这里的id主键假定是uuid):

```
public class UserDaoMapper {

    public String saveAll(Map map) {
        List<User> userList = (List<User>) map.get("user");
        StringBuilder sb = new StringBuilder();
        sb.append("insert into user");
        sb.append("(id,user_name,age)");
        sb.append("values");
        MessageFormat mf = new MessageFormat("uuid(),#'{'user[{0}].id'}',#'{'user[{0}].userName'}',#'{'user[{0}].age'}'");
        for (int i = 0; i < userList.size(); i++) {
            sb.append("(");
            sb.append(mf.format(new Object[]{i}));
            sb.append(")");
            if (i < userList.size() - 1) {
                sb.append(",");
            }
        }
        return sb.toString();
    }
}
```

- 方法二：

```
    @Insert({
            "<script>",
            "insert into user(id,user_name,age)",
            "values",
            "<foreach item='item' collection='list' separator=',' index='index'>",
            "(uuid(),#{item.userName},#{item.age})",
            "</foreach>",
            "</script>"
    })
    void saveAll(@Param("list") List<User> list);
```
>方法二的这种方式其实就是简化的xml形式，格式基本与xml完全一致，只是看着简介明朗了许多。


5、编写测试单元

```java
@RunWith(SpringRunner.class)
@SpringBootTest
//@MapperScan("com.neo.mapper")
public class BootDemoMybatisApplicationTests {
    /* 注入mapper */
    @Autowired
    private UserMapper userMapper;
    /**测试mybatis配置是否完成*/
    @Test
    public void testInsert() throws Exception()
    {
        User user = new User();
        user.setUsername( "这是测试数据 Name" );
        user.setAge( 100 );
        userMapper.inser(user);
        Assert.assertEquals(1, userMapper.getAll().size());
    }
    
    @Test
	public void testQuery() throws Exception {
		List<User> users = userMapper.getAll();
		System.out.println(users.toString());
	}
    
    @Test
	public void testUpdate() throws Exception {
		User user = userMapper.getOne(l);
		System.out.println(user.toString());
		user.setUsername("neo");
		userMapper.update(user);
		Assert.assertTrue(("neo".equals(userMapper.getOne(l).setUsername())));
	}
}
```

6、扩展

mybatis支持SQL语句构建器类，可以让sql语句看的更舒服：
```
String sql = "SELECT P.ID, P.USERNAME, P.PASSWORD, P.FULL_NAME, "
"P.LAST_NAME,P.CREATED_ON, P.UPDATED_ON " +
"FROM PERSON P, ACCOUNT A " +
"INNER JOIN DEPARTMENT D on D.ID = P.DEPARTMENT_ID " +
"INNER JOIN COMPANY C on D.COMPANY_ID = C.ID " +
"WHERE (P.ID = A.ID AND P.FIRST_NAME like ?) " +
"OR (P.LAST_NAME like ?) " +
"GROUP BY P.ID " +
"HAVING (P.LAST_NAME like ?) " +
"OR (P.FIRST_NAME like ?) " +
"ORDER BY P.ID, P.FULL_NAME";

###可以用构造器进行改造：
private String selectPersonSql() {
  return new SQL() {{
    SELECT("P.ID, P.USERNAME, P.PASSWORD, P.FULL_NAME");
    SELECT("P.LAST_NAME, P.CREATED_ON, P.UPDATED_ON");
    FROM("PERSON P");
    FROM("ACCOUNT A");
    INNER_JOIN("DEPARTMENT D on D.ID = P.DEPARTMENT_ID");
    INNER_JOIN("COMPANY C on D.COMPANY_ID = C.ID");
    WHERE("P.ID = A.ID");
    WHERE("P.FIRST_NAME like ?");
    OR();
    WHERE("P.LAST_NAME like ?");
    GROUP_BY("P.ID");
    HAVING("P.LAST_NAME like ?");
    OR();
    HAVING("P.FIRST_NAME like ?");
    ORDER_BY("P.ID");
    ORDER_BY("P.FULL_NAME");
  }}.toString();
}
```

[查看更多属性](http://www.mybatis.org/mybatis-3/zh/java-api.html)


### XML配置文件形式

1、在 application.propertis 中添加配置:

```vim
mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
```

指定了mybatis基础配置文件和实体类映射文件的地址

2、编写 mybatis-config.xml

```xml
<configuration>
	<typeAliases>
		<typeAlias alias="Integer" type="java.lang.Integer" />
		<typeAlias alias="Long" type="java.lang.Long" />
		<typeAlias alias="HashMap" type="java.util.HashMap" />
		<typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap" />
		<typeAlias alias="ArrayList" type="java.util.ArrayList" />
		<typeAlias alias="LinkedList" type="java.util.LinkedList" />
	</typeAliases>
</configuration>
```

这里也可以添加一些mybatis基础的配置


3、编写Mapper接口

```java
public interface UserMapper {

    public List<User> findAll();

    public void inser( User user );

    public void delete( Integer uid );
}
```

4、编写UserMapper.xml

```xml
<mapper namespace="com.neo.mapper.UserMapper">
     <resultMap id="BaseResultMap" type="com.neo.pojo.User">
         <id property="uid" column="id" jdbcType="BIGINT"></id>
         <result property="username" column="name" jdbcType="VARCHAR"></result>
         <result property="age" column="age" jdbcType="BIGINT"></result>
         <!-- <result column="user_sex" property="userSex" javaType="com.neo.enums.UserSexEnum"/>-->
     </resultMap>
     
     <sql id="Base_Column_List" >
        id, name, age
     </sql>
    
     <select id="findAll" resultMap="BaseResultMap">
          SELECT <include refid="Base_Column_List" /> FROM user
     </select>
     
     <insert id="inser" parameterType="com.neo.pojo.User">
          insert into user values (#{username},#{age})
     </insert>
     
     <delete id="delete" parameterType="java.lang.Integer">
          delete from user where id = #{id}
     </delete>
</mapper>
```




### 多数据源配置

1、 application.propertis 中配置

```
mybatis.config-locations=classpath:mybatis/mybatis-config.xml

spring.datasource.test1.driverClassName = com.mysql.jdbc.Driver
spring.datasource.test1.url = jdbc:mysql://localhost:3306/test1?useUnicode=true&characterEncoding=utf-8
spring.datasource.test1.username = root
spring.datasource.test1.password = root

spring.datasource.test2.driverClassName = com.mysql.jdbc.Driver
spring.datasource.test2.url = jdbc:mysql://localhost:3306/test2?useUnicode=true&characterEncoding=utf-8
spring.datasource.test2.username = root
spring.datasource.test2.password = root
```

一个test1库和一个test2库，其中test1位主库，在使用的过程中必须指定主库，不然会报错。

2、数据源配置

```
@Configuration
@MapperScan(basePackages = "com.neo.mapper.test1", sqlSessionTemplateRef  = "test1SqlSessionTemplate")
public class DataSource1Config {

    @Bean(name = "test1DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.test1")
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "test1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test1DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/test1/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "test1TransactionManager")
    @Primary
    public DataSourceTransactionManager testTransactionManager(@Qualifier("test1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "test1SqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("test1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

最关键的地方就是这块了，一层一层注入,首先创建DataSource，然后创建SqlSessionFactory再创建事务，最后包装到SqlSessionTemplate中。其中需要指定分库的mapper文件地址，以及分库dao层代码

### 注意

1、Entity中不映射成列的字段得加`@Transient` 注解