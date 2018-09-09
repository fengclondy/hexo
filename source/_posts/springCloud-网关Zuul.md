---
layout: post
title: springCloud--网关zuul
date: 2018-07-16 01:10:35
tags: springCloud
categories: springCloud
---

参考：https://blog.csdn.net/tianyaleixiaowu/article/details/77893822

### 服务过滤

在服务网关中定义过滤器只需要继承ZuulFilter抽象类实现其定义的四个抽象函数就可对请求进行拦截与过滤。

例子:定义了一个Zuul过滤器，实现了在请求被路由之前检查请求中是否有accessToken参数，若有就进行路由，若没有就拒绝访问，返回401 Unauthorized错误。

```java

public class AccessFilter extends ZuulFilter  {

    private static Logger log = LoggerFactory.getLogger(AccessFilter.class);

    @Override
    public String filterType() {
        return "pre"; //枚举值：pre, routing, post, error
    }

    @Override
    public int filterOrder() {
        return 0; //优先级， 0是最高优先级即最先执行
    }

    @Override
    public boolean shouldFilter() {
        return true; //写逻辑，是否需要执行过滤。true会执行run函数，false不执行run函数
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));

        Object accessToken = request.getParameter("accessToken");
        if(accessToken == null) { 
            log.warn("access token is empty");
            //过滤该请求，不往下级服务去转发请求，到此结束
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            ctx.setResponseBody("{\"result\":\"accessToken为空!\"}");
            ctx.getResponse().setContentType("text/html;charset=UTF-8");
            return null;
        }
         //如果有token，则进行路由转发
        log.info("access token ok");
        //这里return的值没有意义，zuul框架没有使用该返回值
        return null;
    }

}
```


`filterType`：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
- pre：可以在请求被路由之前调用
- routing：在路由请求时候被调用
- post：在routing和error过滤器之后被调用
- error：处理请求时发生错误时被调用
`filterOrder`：通过int值来定义过滤器的执行顺序
`shouldFilter`：返回一个boolean类型来判断该过滤器是否要执行，所以通过此函数可实现过滤器的开关。
`run`：过滤器的具体逻辑。需要注意，这里我们通过ctx.setSendZuulResponse(false)令zuul过滤该请求，不对其进行路由，然后通过ctx.setResponseStatusCode(401)设置了其返回的错误码，当然也可通过ctx.setResponseBody(body)对返回body内容进行编辑等。



在启动类中，配置：
```java
@EnableZuulProxy
@SpringCloudApplication
public class Application {

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}

	@Bean
	public AccessFilter accessFilter() {
		return new AccessFilter();
	}

}
```


对比传统路由：
- 单实例：
```properties
#route rule
zuul.routes.part-1-website.path=/part-1-website/**
zuul.routes.part-1-website.url=http://localhost:1109/
```
说明：以上规则即说明，对当前website应用的所有以`/part-1-website`开始的请求路径全部转发到`http://localhost:1109上`；

- 多实例：
```properties
zuul.routes.part-1-website.path=/part-1-website/**
zuul.routes.part-1-website.serviceId=website
website.ribbon.listOfServers=http://localhost:1108/,http://localhost:1109/
```
注：与单实例不同就是：通过serviceId和listOfServers来确定path和url的对应关系；



#### 限流配置
1、引入提供分布式限流策略的扩展依赖：
```xml
<dependency>
    <groupId>com.marcosbarbero.cloud</groupId>
    <artifactId>spring-cloud-zuul-ratelimit</artifactId>
    <version>1.3.4.RELEASE</version>
</dependency>
```

2、添加配置：
支持的存储方式：
- `InMemoryRateLimiter` - 使用 ConcurrentHashMap作为数据存储
- `ConsulRateLimiter` - 使用 Consul 作为数据存储
- `RedisRateLimiter` - 使用 Redis 作为数据存储
- `SpringDataRateLimiter` - 使用 数据库 作为数据存储

这里采用Redis作为数据储存：
```yaml
zuul:
  ratelimit:
    key-prefix: your-prefix   #对应用来标识请求的key的前缀
    enabled: true 
    repository: REDIS  #对应存储类型（用来存储统计信息）
    behind-proxy: true    #代理之后

    #default-policy:  #可选 - 针对所有的路由配置的策略，除非特别配置了policies
    #  limit: 10 #可选 - 每个刷新时间窗口对应的请求数量限制
    #  quota: 1000 #可选-  每个刷新时间窗口对应的请求时间限制（秒）
    #  refresh-interval: 60 # 刷新时间窗口的时间，默认值 (秒)
    #  type: #可选 限流方式
    #    - user
    #    - origin
    #    - url

    policies:  #特定路由
      myServiceId:   #特定的路由，改成服务的Id号即可
        limit: 10
        quota: 20
        refresh-interval: 30
        type:
          - user
```
- limit 单位时间内允许访问的个数
- quota 单位时间内允许访问的总时间（统计每次请求的时间综合）
- refresh-interval 单位时间设置

以上配置意思是：30秒内允许10个访问，或者要求总请求时间小于20秒

>注意：1、可以使用Spring Boot Actuator 提供的服务状态，动态设置限流开关
2、如果你的项目整合 Shiro 或者 Spring Security 安全框架，那么会自动维护request域UserPrincipal，
如果是自己的框架，请登录成功后维护request域UserPrincipal，才能使用用户粒度的限流，未登录默认是：anonymous。
具体代码实现可以看 DefaultRateLimitKeyGenerator,type为USER的实现

-------------------------  摘录 ---------------------------

### 另一种jwt
1、
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
```
2、
```java
protected void doFilterInternal(
    HttpServletRequest request,
    HttpServletResponse response,
    FilterChain chain) throws ServletException, IOException {
    String authHeader = request.getHeader(this.tokenHeader);
    if (authHeader != null && authHeader.startsWith(tokenHead)) {
        String authToken = authHeader.substring(tokenHead.length()); // The part after "Bearer "
        String username = jwtTokenUtil.getUsernameFromToken(authToken);
        logger.info("checking authentication " + username);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);

            if (jwtTokenUtil.validateToken(authToken, userDetails)) {
                UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(
                        request));
                logger.info("authenticated user " + username + ", setting security context");
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }
    }

    chain.doFilter(request, response);
}
```
3、
```java
protected void configure(HttpSecurity httpSecurity) throws Exception {
httpSecurity
    // 由于使用的是JWT，我们这里不需要csrf
    .csrf().disable()

    .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()

    // 基于token，所以不需要session
    .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()

    .authorizeRequests()
    //.antMatchers(HttpMethod.OPTIONS, "/**").permitAll()

    // 允许对于网站静态资源的无授权访问
    .antMatchers(
            HttpMethod.GET,
            "/",
            "/*.html",
            "/favicon.ico",
            "/**/*.html",
            "/**/*.css",
            "/**/*.js"
    ).permitAll()
    .antMatchers("/auth/**").permitAll()
    .anyRequest().authenticated();
// 添加JWT filter
httpSecurity
        .addFilterBefore(authenticationTokenFilterBean(), UsernamePasswordAuthenticationFilter.class);
// 禁用缓存
httpSecurity.headers().cacheControl();
}
```
