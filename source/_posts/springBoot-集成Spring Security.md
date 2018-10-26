---
layout: post
title: Spring Boot集成Spring Security
date: 2018-02-20 03:26:57
tags: springBoot
categories: springBoot
---


### 初阶 Security： 默认认证用户名密码
1、在 pom.xml 中添加spring-boot-starter-security依赖
```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
启动项目，访问网页即会出现弹出框。  
>security默认的用户名是user, 默认密码是应用启动的时候，通过UUID算法随机生成的。默认的role是"USER"

2、当然，为了方便，可以在application.properties配置你的用户名密码，例如
```properties
# security
security.user.name=admin
security.user.password=admin
```

<!-- more -->

### 中阶 Security：内存用户名密码认证

下面进行定制用户名密码（多用户）
```java
package com.favorites.security;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

/**
 * Created by george on 2018/2/17.
 */

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class WebSecurity extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
    }
    //WebSecurityConfigurerAdapter共有三个configure方法
    //configure(WebSecurity web) 通过重载，配置Spring Security的Filter链
    //configure(HttpSecurity http) 通过重载，配置如何通过拦截器保护请求
    //configure(AuthenticationManagerBuilder auth) 通过重载，配置user-detail服务

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .inMemoryAuthentication()
                .withUser("root")
                .password("root")
                .roles("USER")

                .and()
                .withUser("admin").password("admin")
                .roles("ADMIN", "USER")

                .and()
                .withUser("user").password("user")
                .roles("USER");
    }
}
```
简要说明：  
1、代码里配置的用户名与密码优先级比属性里配置的高，会使属性配置失效。  
2、Spring Security提供了Spring EL表达式，允许我们在定义URL路径访问(@RequestMapping)的方法上面添加注解，
 来控制访问权限。`true`，表示有权限; `fasle`，表示无权限。  
3、`@PreAuthorize("hasRole('ADMIN')")` 在方法调用前，通过SpringEL表达式限制方法访问 
  // Spring Security默认的角色前缀是”ROLE_”,使用hasRole方法可切换对应的角色  
  // @PreAuthorize这个注解,当表达式值为true，标识这个方法可以被调用。如果表达式值是false，标识此方法无权限访问
例子：
```java
@PreAuthorize("hasRole('ROLE_ADMIN')")
public void addUser(User user){
    //如果具有权限 ROLE_ADMIN 访问该方法
    ....
}
```
4、`@PostAuthorize`：允许方法调用，但时如果表达式结果为false抛出异常
```java
//returnObject可以获取返回对象user，判断user属性username是否和访问该方法的用户对象的用户名一样。不一样则抛出异常。
@PostAuthorize("returnObject.user.username==principal.username")
public User getUser(int userId){
   //允许进入
...
    return user;    
}
```

5、`@PostFilter`：允许方法调用，但必须按表达式过滤方法结果
```java
//将结果过滤，即选出性别为男的用户
@PostFilter("returnObject.user.sex=='男' ")
public List<User> getUserList(){
    //允许进入
    ...
    return user;    
}
```

#### 获取登录用户的信息
代码参考：
```java
Authentication authentication = (Authentication) httpServletRequest.getUserPrincipal();
    Object principal = authentication.getPrincipal();
    Object userDetails = null;

if (principal instanceof UserDetails) {
    String username = ((UserDetails)principal).getUsername();
} else {
    String username = principal.toString();
}
```

方法参考：https://www.jianshu.com/p/ac42f38baf6e

### 进阶 Security： 用数据库存储用户和角色，实现安全认证
>简单设计两个用户角色：USER，ADMIN，设计思路：
首页/ : 所有人可访问
登录页 /login: 所有人可访问
普通用户权限页 /httpapi, /httpsuite: 登录后的用户都可访问
管理员权限页 /httpreport ： 仅管理员可访问
无权限提醒页： 当一个用户访问了其没有权限的页面，我们使用全局统一的异常处理页面提示。

1、数据库层设计：新建三张表User,Role,UserRole（此处省略）


参考：https://www.jianshu.com/p/6c87b8304fc9



### Spring security实现第三方登录授权服务，使用Oauth2.0协议

#### github授权登录
在github中，使用的是授权码模式，在这种模式中，客户端需要引导用户重定向请求到Github的OAuth认证接口,带上在授权服务器申请的客户端 client_id,配置的回调地址 redirect_uri ,申请授权的scope,你也可以带上状态值state,服务器会在回调中原封不动地返回你这个值，这一步将会换取前面我们说到的授权码code。

在[页面](https://github.com/settings/developers)注册一个OAuth应用，填写相应的应用名称，主页url和回调地址:
![](http://p2jr3pegk.bkt.clouddn.com/springBoot-s02.png)

完后注册后，可获得客户端的Client ID和Client Secret。

```java
@Controller
@RequestMapping("/OAuth")
public class OAuthController {
	/**
	 * 1.访问用户登录的验证接口
	 * https://github.com/login/oauth/authorize?client_id=xxxxxxxxxxxxxxxxxx&scope=user,public_repo
	 * 2.访问上面接口后会github会让其跳转到你预定的url(Authorization callback URL)，并且带上code参数,例如
	 * http://localhost:8080/callback?code=****************
	 * 3.然后，开发者可以通过code,client_id以及client_secret这三个参数获取用户的access_token即用户身份标识，请求如下
	 * https://github.com/login/oauth/access_token?client_id=xxxxxxxxxxxxxxxxxxx&client_secret=xxxxxxxxxxxxxxxxx&code=xxxxxxxxxxxxxxxxxxx
	 * 这样就会返回access_token,如下
	 * access_token=xxxxxxxxxxxxxxxxxxxxxxxxx&scope=public_repo%2
	 * Cuser&token_type=bearer 4. 这样我们就可以用这个access_token来获取用户的信息
	 * https://api.github.com/user?access_token=xxxxxxxxxxxxxxxxxxxxxxxxx
	 * 
	 */

	@Autowired
	private UserRepository userRepository;
	@Autowired
	private RoleRepository roleRepository;

	@Autowired
	protected AuthenticationManager authenticationManager;

	private static final String PROTECTED_RESOURCE_URL = "https://api.github.com/user";
	//Github上申请的appId
	private String appId = e7a2d0c08578e2a2b110；
	//Github上申请的appSecret
	private String appSecret = 8da3a2eaf9d246ef7b8e4aa0b0e867cfc7a8dcfe; 
	//Github登录成功之后的回调系统地址
	private String callbackUrl = http://127.0.0.1:8080/OAuth/callback/getOAuth ；
	//系统跳转到Github认证登录地址
	private String redrictUrl = https://github.com/login/oauth/authorize?client_id=e7a2d0c08578e2a2b110&scope=user,public_repo ；
	
	//前台点击使用github账号登录，返回的重定向url
	@RequestMapping(value = "/authLogin", method = RequestMethod.GET)
	public void authLogin(HttpServletRequest request, HttpServletResponse response) {
		try {
			response.sendRedirect(redrictUrl);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	@RequestMapping(value = "/callback/getOAuth", method = RequestMethod.GET)
	public String getOAuth(@RequestParam(value = "code", required = true) String code, Model model,
			HttpServletRequest request, HttpServletResponse response) {
		String secretState = "secret" + new Random().nextInt(999_999);
		OAuth20Service service = new ServiceBuilder(appId)
				.apiSecret(appSecret).state(secretState)
				.callback(callbackUrl).build(GitHubApi.instance());
		OAuth2AccessToken accessToken = null;
		GithubUser githubUser = null;
		try {
			accessToken = service.getAccessToken(code);
			final OAuthRequest oAuthRequest = new OAuthRequest(Verb.GET, PROTECTED_RESOURCE_URL);
			service.signRequest(accessToken, oAuthRequest);
			final Response oAuthresponse = service.execute(oAuthRequest);
			githubUser = JSON.parseObject(oAuthresponse.getBody(), new TypeReference<GithubUser>() {});
			model.addAttribute("user", oAuthresponse.getBody());
			System.out.println(oAuthresponse.getBody());
		} catch (IOException e) {
			model.addAttribute("error", "github登录失败！");
			return "login";
		} catch (InterruptedException e) {
			model.addAttribute("error", "github登录失败！");
			return "login";
		} catch (ExecutionException e) {
			model.addAttribute("error", "github登录失败！");
			return "login";
		} catch (OAuthException e) {
			model.addAttribute("error", "github登录失败！");
			return "login";
		}
		// TODO
		// 1、判断是不是第一次授权登录,新增用户写进数据库，给一个默认角色
		/*
		 * if(第一次授权登录){ 
		 * 新增用户写进数据库，给一个默认角色，user表新增字段第三方登录方式及第三方用户id
		 * 新增userDetail表来存储用户详细信息 
		 * }else{ 
		 * 	根据第三方用户id获取以前存进库里的信息登录 
		 * }
		 */
		List<RoleEntity> roles = new ArrayList<RoleEntity>();
		roles.add(roleRepository.findOne((long) 2));//普通用户
		UserEntity userEntity = new UserEntity();
		userEntity.setUsername(githubUser.getLogin());
		userEntity.setPassword("123456");
		userEntity.setId((long) 3);
		userEntity.setRoles(roles);
		userEntity = userRepository.saveAndFlush(userEntity);
		List<GrantedAuthority> authorities = new ArrayList<>();
		authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
		Authentication authentication = new UsernamePasswordAuthenticationToken(userEntity.getUsername(), userEntity.getPassword(), authorities);
		//List<GrantedAuthority> authorities = new ArrayList<>();
		//authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
		//authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
		//Authentication authentication = new UsernamePasswordAuthenticationToken("admin", "123456", authorities);
		// 将token传递给Authentication进行验证
		Authentication result = authenticationManager.authenticate(authentication);
		SecurityContextHolder.getContext().setAuthentication(result);
		UserEntity user = (UserEntity) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
		List<RoleEntity> roleList = user.getRoles();
		Set<ResourceEntity> resourceList = new HashSet<ResourceEntity>();
		String roles1 = "";
		for (RoleEntity role : roleList) {
			roles1 += role.getName() + ",";
			resourceList.addAll(role.getResources());
		}
		roles1 = roles1.substring(0, roles1.length() - 1);
		request.getSession().setAttribute("roles", roles1);
		System.out.println("====================="+roles1);
		for(ResourceEntity r: resourceList) {
			System.out.println(r.getName());
		}
		System.out.println(ResourceUtil.format(new ArrayList<>(resourceList)));
		model.addAttribute("resourceList", ResourceUtil.format(new ArrayList<>(resourceList)));
		return "github_success";
	}
}
```
正常的显示页面为:![](http://p2jr3pegk.bkt.clouddn.com/springBoot-s01.png)

![](http://p2jr3pegk.bkt.clouddn.com/springBoot-s03.png)