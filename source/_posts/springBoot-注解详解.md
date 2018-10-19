---
layout: post
title: springBoot--注解大全
date: 2018-01-24 12:35:57
tags: springBoot
categories: springBoot
---

### 1、注解(annotations)列表
* `@SpringBootApplication`：等同于@ComponentScan、@Configuration和@EnableAutoConfiguration三个注解。其中@
让spring Boot扫描到Configuration类并把它加入到程序上下文。

* `@RestController`:注解是@Controller和@ResponseBody的合集,表示这是个控制器bean,并且是将函数的返回值直 接填入HTTP响应体中,是REST风格的控制器。

* `@Value`：注入Spring boot application.properties配置的属性的值。

* `@RepositoryRestResourcepublic`:配合spring-boot-starter-data-rest使用。

* `@CrossOrigin`:跨域访问


### 2、JPA注解

* `@Entity：@Table(name="")`：表明这是一个实体类。一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略
* `@Valid`：验证数据校验（对象）
* `@MappedSuperClass`:用在确定是父类的entity上。父类的属性子类可以继承。
* `@NoRepositoryBean`:一般用作父类的repository，有这个注解，spring不会去实例化该repository。
* `@Column`：如果字段名与列名相同，则可以省略。
* `@Id`：表示该属性为主键。
* `@GeneratedValue(strategy=GenerationType.SEQUENCE,generator="repair_seq")`：表示主键生成策略是sequence（可以为Auto、IDENTITY、native等，Auto表示可在多个数据库间切换），指定sequence的名字是repair_seq。
* `@SequenceGeneretor(name="repair_seq",sequenceName="seq_repair",allocationSize=1)`：name为sequence的名称，以便使用，sequenceName为数据库的sequence名称，两个名称可以一致。
* `@Transient`：表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic。@Basic(fetch=FetchType.LAZY)：标记可以指定实体属性的加载方式
* `@JsonIgnore`：作用是json序列化时将Java bean中的一些属性忽略掉,序列化和反序列化都受影响。
* `@JoinColumn（name="loginId"）`:一对一：本表中指向另一个表的外键。一对多：另一个表指向本表的外键。
* `@OneToOne、@OneToMany、@ManyToOne`：对应hibernate配置文件中的一对一，一对多，多对一。

<!-- more -->

### 3、Spring MVC相关注解

* `@Controller`：用于定义控制器类。

* `@Service：` : 修饰service层的组件.

* `@Repository`：修饰DAO或者repositories。

* `@RequestMapping`：提供路由信息，负责URL到Controller中的具体函数的映射。该注解有六个属性： 
>params:指定request中必须包含某些参数值是，才让该方法处理。 
>headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。 
>value:指定请求的实际地址，指定的地址可以是URI Template 模式 
>method:指定请求的method类型， GET、POST、PUT、DELETE等 
>consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html; 
>produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回

当然可以`用@GetMapping`、`@PostMapping`等替代。如：`@GetMapping(value = {"/info", "/info/{username}"})`
* `@ComponentScan`: 组件扫描，可自动发现和装配一些Bean，并注入到IOC容器中。如 

@Component、@Service、 @Repository、 @Controller、@Entity 等等

* `@Component`：定义Spring管理Bean。它的扩展类有：@Repository、@Service、@Controller

* `@Configuration`: 等同于spring的XML配置文件中的 ``<beans>``，作用为：配置 spring 容器(应用上下文)；使用Java代码可以检查类型安全。

* `@EnableAutoConfiguration`: 自动配置。

* `@RequestParam`：用在方法的参数前面

* `@RequestBody`、`@ResonseBody`、`@RathVariable`

* `@Autowired`:自动导入依赖的bean。

* `@PathVariable`:获取参数,与`@RequestParam`区别在于，能够自动识别处模板。

* `@Component`:可配合CommandLineRunner使用，在程序启动后执行一些基础任务。

* `@Bean`：用@Bean标注方法等价于XML中配置的bean。作用为：注册 bean 对象

* `@Qualifier`：当有多个同一类型的Bean时，可以用@Qualifier("name")来指定。

* `@Inject`：等价于默认的@Autowired，只是没有required属性。

* `@JsonBackReference`:解决嵌套外链问题。

* `@Import`：用来导入其他配置类中的bean属性。

* `@ImportResource`：用来加载xml配置文件。

* `@Resource(name="name",type="type")`：与@Autowired类似。

* `@Jsonfield注解`: 设置方法或者属性的setter和getter方法。

* `@Pointcut`: 拦截的切入点方法，注解的在方法级别之上，但是不执行方法体，只表示切入点的入口。

* `@Around`: 判断是否执行以上的拦截,后面跟的值为申明切入点的方法名，而该注解申明的方法的第一个参数必须ProceedingJoinPoint。

* `@Cacheable`: 支持缓存，指定三个属性，value、key和condition。

* `@CachePut`: 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存，和 @Cacheable 不同的是，它每次都会触发真实方法的调用.

* `@CacheEvict`: 主要针对方法配置，能够根据一定的条件对缓存进行清空.


### 4、全局异常处理

* `@ControllerAdvice`：包含@Component。可以被扫描到。统一处理异常。

* `@ExceptionHandler（Exception.class）`：用在方法上面表示遇到这个异常就执行以下方法。

### 5、其他Spring Boot相关

* 1、`@EnableScheduling`: 启用定时任务的配置（启动类）
  与`@Scheduled`（任务类）搭配使用。
  
 >@Scheduled(fixedRate = 5000) ：上一次开始执行时间点之后5秒再执行
 >@Scheduled(fixedDelay = 5000) ：上一次执行完毕时间点之后5秒再执行
 > @Scheduled(initialDelay=1000, fixedRate=5000) ：第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
 > @Scheduled(cron="*/5 * * * * *") ：通过cron表达式定义规则

* 2、`@Async`：实现异步调用
* 3、`@Aspect`:切面编程。与@Pointcut、@Before、@After、@AfterReturning等配合使用。
* 4、`@EnableCaching`:开启缓存功能（启动类）。与`@CacheConfig`和`@Cacheable`一起使用（配置类）。
    缓存配置这里不叙述，可自行百度，推荐使用redis或ecache，[EhCache学习](http://blog.csdn.net/vbirdbest/article/details/72763048)
* 5、`@PostConstruct`：在构造函数执行后执行； `@PreDestory`：在Bean销毁之前执行。
* 6、`@EnableAsync`:开启异步任务支持 ；`@Async`:注解该方法/类为异步方法。
* 7、`@EnableScheduling`:开启定时任务； `@Scheduled(fixedDelay = 1000*60)`:定时任务，60s执行一次。
* 8、`@Conditional`：条件匹配注解。
* 9、`@ControllerAdvice`：将全局配置放在同一位置（类上）。友联此注解，可用 `@ExceptionHandler`：全局处理异常。`@ModelAttribute`:将键值对绑定到model里，让全局@RequestMapping都可获得该值。
* 10、`@CachePut`：缓存新增的或更新数据到缓存; `@CacheEvict`:缓存删除后的数据。`@Cacheable`：缓存key的数据。
* 11、`@ConfigurationProperties`：作用就是通过它可以把properties或者yml配置中的配置文件直接转成对象。@ConfigurationProperties(prefix = "sms")申明配置的前缀为sms。
* 12、`@PropertySource("classpath:global.properties")`: 等价于`@ConfigurationProperties`，前者需要指明具体的文件路径和后缀，后者可以自动读取.yml和.properties文件。配置文件中的参数若有分层结构，只需加上对应的前缀即可。
* 13、`@RefreshScope`： 动态刷新配置。当boot中的properties值改变，SpringCloud触发，主要用于更改了配置文件，实现配置文件的动态刷新。
* 14、`@ConditionalOnExpression()`：在特定情况下使用相关配置或者实例化bean。
* 15、`@ConfigurationProperties(prefix="xxx")`：把使用统一前缀的同类的配置信息
* 16、`@Primary`：自动装配时当出现多个Bean候选者时，被注解为@Primary的Bean将作为首选者，否则将抛出异常
* 17、`@EnableAuthorizationServer`：配置授权服务OAuth2
* 18、`@SpringCloudApplication`：该注解，整合了@SpringBootApplication、@EnableDiscoveryClient、@EnableCircuitBreaker
* 19、`@PreAuthorize`：配置接口权限校验，先验证
* 20、`@EnableGlobalMethodSecurity(prePostEnabled = true)`：在WebSecurityConfigurerAdapter 的继承类上加，与19对应使用

### 6、SpringCloud 相关
* 1、`@SpringCloudApplication`：包括@SpringBootApplication、@EnableDiscoveryClient、@EnableCircuitBreaker，分别是SpringBoot注解、注册服务中心Eureka注解、断路器注解。

