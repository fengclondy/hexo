---
layout: post
title: sprinBoot-自定义注解
date: 2018-07-27 03:23:16
tags: springBoot
categories: springBoot
---

### 自定义权限注解

1、定义一个自定义注解类
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Access {

    String[] value() default {};
    String[] authorities() default {};
    String[] roles() default {};
}
```
参数说明：
>`@Target`注解是标注这个类它可以标注的位置.   
`@Retention`注解表示的是本注解(标注这个注解的注解保留时期).   
`@Documented`是否生成文档的标注, 也就是生成接口文档是, 是否生成注解文档.  

<!-- more -->

常用的元素类型及保留时间：

```java
public enum ElementType {
    // TYPE类型可以声明在类上或枚举上或者是注解上
    TYPE,
    // FIELD声明在字段上
    FIELD,
    // 声明在方法上
    METHOD,
    // 声明在形参列表中
    PARAMETER,
    // 声明在构造方法上
    CONSTRUCTOR,
    // 声明在局部变量上
    LOCAL_VARIABLE,
    /** Annotation type declaration */
    ANNOTATION_TYPE,
    /** Package declaration */
    PACKAGE,
    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,
    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

```java
public enum RetentionPolicy {
    // 源代码时期
    SOURCE,
    // 字节码时期, 编译之后
    CLASS,
    // 运行时期, 也就是一直保留, 通常也都用这个
    RUNTIME
}
```

2、在方法上配置权限注解
```java
@RestController
public class HelloController {

    @RequestMapping(value = "/admin", produces = MediaType.APPLICATION_JSON_UTF8_VALUE, method = RequestMethod.GET)
    // 配置注解权限, 允许身份为admin, 或者说允许权限为admin的人访问
    @Access(authorities = {"admin"})
    public String hello() {
        return "Hello, admin";
    }
}

```

3、编写权限逻辑
```java
// 自定义一个权限拦截器, 继承HandlerInterceptorAdapter类
public class AuthenticationInterceptor extends HandlerInterceptorAdapter {

    // 在调用方法之前执行拦截
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 将handler强转为HandlerMethod, 前面已经证实这个handler就是HandlerMethod
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        // 从方法处理器中获取出要调用的方法
        Method method = handlerMethod.getMethod();
        // 获取出方法上的Access注解
        Access access = method.getAnnotation(Access.class);
        if (access == null) {
        // 如果注解为null, 说明不需要拦截, 直接放过
            return true;
        }
        if (access.authorities().length > 0) {
            // 如果权限配置不为空, 则取出配置值
            String[] authorities = access.authorities();
            Set<String> authSet = new HashSet<>();
            for (String authority : authorities) {
            // 将权限加入一个set集合中
                authSet.add(authority);
            }
            // 这里我为了方便是直接参数传入权限, 在实际操作中应该是从参数中获取用户Id
            // 到数据库权限表中查询用户拥有的权限集合, 与set集合中的权限进行对比完成权限校验
            String role = request.getParameter("role");
            if (StringUtils.isNotBlank(role)) {
                if (authSet.contains(role)) {
                // 校验通过返回true, 否则拦截请求
                    return true;
                }
            }
        }
        // 拦截之后应该返回公共结果, 这里没做处理
        return false;
    }
}
```

### 自定义Swagger 接口注解实现
>这里以定义一个全局的 common-swagger 包为例，进行说明：

1、添加依赖：
```xml
<!--swagger 依赖-->
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.9.2</version>
</dependency>
```

2、增加一个自定义注解的相关属性。
```java
@Data
@ConfigurationProperties("swagger")
public class SwaggerProperties {
	/**
	 * swagger会解析的包路径
	 **/
	private String basePackage = "";
	/**
	 * swagger会解析的url规则
	 **/
	private List<String> basePath = new ArrayList<>();
	/**
	 * 在basePath基础上需要排除的url规则
	 **/
	private List<String> excludePath = new ArrayList<>();
	/**
	 * 标题
	 **/
	private String title = "";
	/**
	 * 描述
	 **/
	private String description = "";
	/**
	 * 版本
	 **/
	private String version = "";
	/**
	 * 许可证
	 **/
	private String license = "";
	/**
	 * 许可证URL
	 **/
	private String licenseUrl = "";
	/**
	 * 服务条款URL
	 **/
	private String termsOfServiceUrl = "";
	/**
	 * host信息
	 **/
	private String host = "";
	/**
	 * 联系人信息
	 */
	private Contact contact = new Contact();
	/**
	 * 全局统一鉴权配置
	 **/
	private Authorization authorization = new Authorization();

	@Data
	@NoArgsConstructor
	public static class Contact {
		/**
		 * 联系人
		 **/
		private String name = "";
		/**
		 * 联系人url
		 **/
		private String url = "";
		/**
		 * 联系人email
		 **/
		private String email = "";
	}

	@Data
	@NoArgsConstructor
	public static class Authorization {
		/**
		 * 鉴权策略ID，需要和SecurityReferences ID保持一致
		 */
		private String name = "";

		/**
		 * 需要开启鉴权URL的正则
		 */
		private String authRegex = "^.*$";

		/**
		 * 鉴权作用域列表
		 */
		private List<AuthorizationScope> authorizationScopeList = new ArrayList<>();

		private List<String> tokenUrlList = new ArrayList<>();
	}

	@Data
	@NoArgsConstructor
	public static class AuthorizationScope {
		/**
		 * 作用域名称
		 */
		private String scope = "";
		/**
		 * 作用域描述
		 */
		private String description = "";
	}
}
```

3、添加自定义自动配置：
```java
@Configuration
@EnableSwagger2
@EnableAutoConfiguration
public class SwaggerAutoConfiguration {
	/**
	 * 	默认的排除路径，排除Spring Boot默认的错误处理路径和端点
	 */
	private static final List<String> DEFAULT_EXCLUDE_PATH = Arrays.asList("/error","/actuator/**");
	private static final String BASE_PATH = "/**";

	@Bean
	@ConditionalOnMissingBean
	public SwaggerProperties swaggerProperties() {
		return new SwaggerProperties();
	}

	@Bean
	public Docket api(SwaggerProperties swaggerProperties) {
		// base-path处理
		if (swaggerProperties.getBasePath().isEmpty()) {
			swaggerProperties.getBasePath().add(BASE_PATH);
		}
		//noinspection unchecked
		List<Predicate<String>> basePath = new ArrayList();
		swaggerProperties.getBasePath().forEach(path -> basePath.add(PathSelectors.ant(path)));

		// exclude-path处理
		if (swaggerProperties.getExcludePath().isEmpty()) {
		swaggerProperties.getExcludePath().addAll(DEFAULT_EXCLUDE_PATH);
		}
		List<Predicate<String>> excludePath = new ArrayList<>();
		swaggerProperties.getExcludePath().forEach(path -> excludePath.add(PathSelectors.ant(path)));

		//noinspection Guava
		return new Docket(DocumentationType.SWAGGER_2)
			.host(swaggerProperties.getHost())
			.apiInfo(apiInfo(swaggerProperties)).select()
			.apis(RequestHandlerSelectors.basePackage(swaggerProperties.getBasePackage()))
			.paths(Predicates.and(Predicates.not(Predicates.or(excludePath)), Predicates.or(basePath)))
			.build()
			.securitySchemes(Collections.singletonList(securitySchema()))
			.securityContexts(Collections.singletonList(securityContext()))
			.pathMapping("/");
	}

	/**
	 * 配置默认的全局鉴权策略的开关，通过正则表达式进行匹配；默认匹配所有URL
	 *
	 * @return
	 */
	private SecurityContext securityContext() {
		return SecurityContext.builder()
			.securityReferences(defaultAuth())
			.forPaths(PathSelectors.regex(swaggerProperties().getAuthorization().getAuthRegex()))
			.build();
	}

	/**
	 * 默认的全局鉴权策略
	 *
	 * @return
	 */
	private List<SecurityReference> defaultAuth() {
		ArrayList<AuthorizationScope> authorizationScopeList = new ArrayList<>();
swaggerProperties().getAuthorization().getAuthorizationScopeList().forEach(authorizationScope -> authorizationScopeList.add(new AuthorizationScope(authorizationScope.getScope(), authorizationScope.getDescription())));
		AuthorizationScope[] authorizationScopes = new AuthorizationScope[authorizationScopeList.size()];
		return Collections.singletonList(SecurityReference.builder()
			.reference(swaggerProperties().getAuthorization().getName())
			.scopes(authorizationScopeList.toArray(authorizationScopes))
			.build());
	}


	private OAuth securitySchema() {
		ArrayList<AuthorizationScope> authorizationScopeList = new ArrayList<>();
		swaggerProperties().getAuthorization().getAuthorizationScopeList().forEach(authorizationScope -> authorizationScopeList.add(new AuthorizationScope(authorizationScope.getScope(), authorizationScope.getDescription())));
		ArrayList<GrantType> grantTypes = new ArrayList<>();
swaggerProperties().getAuthorization().getTokenUrlList().forEach(tokenUrl -> grantTypes.add(new ResourceOwnerPasswordCredentialsGrant(tokenUrl)));
		return new OAuth(swaggerProperties().getAuthorization().getName(), authorizationScopeList, grantTypes);
	}

	private ApiInfo apiInfo(SwaggerProperties swaggerProperties) {
		return new ApiInfoBuilder()
			.title(swaggerProperties.getTitle())
			.description(swaggerProperties.getDescription())
			.license(swaggerProperties.getLicense())
			.licenseUrl(swaggerProperties.getLicenseUrl())
			.termsOfServiceUrl(swaggerProperties.getTermsOfServiceUrl())
			.contact(new Contact(swaggerProperties.getContact().getName(), swaggerProperties.getContact().getUrl(), swaggerProperties.getContact().getEmail()))
			.version(swaggerProperties.getVersion())
			.build();
	}
}
```

4、自定义注解的注解名称。
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({SwaggerAutoConfiguration.class})
public @interface EnablePigxSwagger2 {
}
```
5、接下来引入上面的common-wagger包，在启动类Application头部加上`@EnablePigxSwagger2`即可，然后在对应的controller接口的方法上，仍然使用自带的一些注解。如：`@ApiOperation`,`@ApiImplicitParam`

6、当然，千万不要忘了全局配置,所有的名称与你定义的属性文件相对应：
```yaml
#swagger公共信息
swagger:
  title: PigX Swagger API
  description: 全宇宙最牛逼的Spring Cloud微服务开发脚手架
  version: 1.6.2
  license: Powered By PigX
  licenseUrl: https://gqsu.top
  terms-of-service-url: https://gqsu.top
  contact:
    name: gqsu
    email: 597009281@qq.com
    url: https://gqsu.top
  authorization:
    authorization-scope-list:
      - scope: 'server'
        description: 'server all'
      - scope: 'read'
        description: 'read all'
      - scope: 'write'
        description: 'access all'
```

### 扩展自动配置 spring.factories
spring.factories这个主要是提供了一个功能，就是自动配置，不需要使用@EnableXXX来开启，也就是说只要你用了springboot，并且依赖了一个jar包，这个jar包就会自动进行初始化。一般存放在resources/META-INF/下：
```shell
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.pig4cloud.pigx.admin.api.feign.fallback.RemoteUserServiceFallbackImpl,\
  com.pig4cloud.pigx.admin.api.feign.fallback.RemoteLogServiceFallbackImpl,\
```
springboot启动的时候就会自动进行扫描。（一般用在common包中，不需要再添加@SpringCloudApplication启动）
