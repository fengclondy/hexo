---
layout: post
title: springCloud-鉴权中心oauth2
date: 2018-08-13 07:01:21
tags: springCloud
categories: springCloud
---

### 基本概念

oauth2 定义了下面四种授权方式：

- 授权码模式（authorization code）
- 简化模式（implicit）
- 密码模式（resource owner password credentials）
- 客户端模式（client credentials）

常见参数：
```
* response_type：表示授权类型，必选项，此处的值固定为"code"

* client_id：表示客户端的ID，必选项

* redirect_uri：表示重定向URI，可选项

* scope：表示申请的权限范围，可选项

* state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。
```

<!-- more -->

过滤器

- ClientCredentialsTokenEndpointFilter： 请求/oauth/token认证客户端。

相关类：
- AuthorizationServerTokenServices：令牌管理。
- AuthorizationCodeServices设置为JdbcAuthorizationCodeServices，使用jdbc进行维护code，将code存储在表oauth_code（框架自带的表结构，在文末会一起给出）。
- TokenGranter ： 接入令牌的接口，提供令牌操作扩展（grant_type参数authorization_code,password,implicit操作等）。

请求端点：
- AuthorizationEndpoint用于为授权请求提供服务。默认网址：/oauth/authorize。
- TokenEndpoint用于服务访问令牌的请求。默认网址：/oauth/token。


### 使用详解
1、权限认证服务器：
```java
/**
 * @author gqsu
 * @date 2018/6/22
 * 认证服务器配置（授权验证配置，继承AuthorizationServerConfigurerAdapter）
 */
@Configuration
@AllArgsConstructor
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
	@Autowired
	private final DataSource dataSource;
	@Autowired
	private final AuthenticationManager authenticationManager;
	@Autowired
	private final RedisConnectionFactory redisConnectionFactory;

    // 配置客户端, 用于client认证
	@Override
	public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        /** 这里使用数据库密码验证，判断是否有权限*/
		JdbcClientDetailsService clientDetailsService = new JdbcClientDetailsService(dataSource);
		//clientDetailsService.setSelectClientDetailsSql(SecurityConstants.DEFAULT_SELECT_STATEMENT);
//clientDetailsService.setFindClientDetailsSql(SecurityConstants.DEFAULT_FIND_STATEMENT);

        String selectSql= "select client_id, CONCAT('{noop}',client_secret) as client_secret, resource_ids, scope, "
		+ "authorized_grant_types, web_server_redirect_uri, authorities, access_token_validity, "
		+ "refresh_token_validity, additional_information, autoapprove from sys_oauth_client_details where client_id = ?"
        clientDetailsService.setSelectClientDetailsSql(selectSql);

        String findSql = "select client_id, CONCAT('{noop}',client_secret) as client_secret, resource_ids, scope, "
		+ "authorized_grant_types, web_server_redirect_uri, authorities, access_token_validity, "
		+ "refresh_token_validity, additional_information, autoapprove from sys_oauth_client_details order by client_id";
        clientDetailsService.setFindClientDetailsSql(findSql);
		
		clients.withClientDetails(clientDetailsService);

        /*    //基于内存存储令牌
            clients.inMemory().withClient("client_1")
                .resourceIds(DEMO_RESOURCE_ID)
                .authorizedGrantTypes("client_credentials", "refresh_token")
                .scopes("all")
                .authorities("client")
                .secret("123456");*/
	}

	@Override
	public void configure(AuthorizationServerSecurityConfigurer oauthServer) throws Exception {
        /** 允许表单验证 */
		oauthServer
			.allowFormAuthenticationForClients()
			.checkTokenAccess("permitAll()");
	}

	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
		endpoints
			.allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST)
			.tokenStore(tokenStore())
			.tokenEnhancer(tokenEnhancer())
			.authenticationManager(authenticationManager)
			.reuseRefreshTokens(false)
			.exceptionTranslator(new PigxWebResponseExceptionTranslator());
	}


    // 初始化TokenStore
	@Bean
	public TokenStore tokenStore() {
        /* 这里采用了redis来存储 */
		RedisTokenStore tokenStore = new RedisTokenStore(redisConnectionFactory);
        //JdbcTokenStore tokenStore = new JdbcTokenStore(dataSource);
        /** 前缀 */
        //tokenStore.setPrefix(SecurityConstants.PIGX_PREFIX + SecurityConstants.OAUTH_PREFIX);
		tokenStore.setPrefix("pig_oauth:");
		return tokenStore;
	}

	@Bean
	public TokenEnhancer tokenEnhancer() {
		return (accessToken, authentication) -> {
			final Map<String, Object> additionalInfo = new HashMap<>(1);
			//additionalInfo.put("license", SecurityConstants.PIGX_LICENSE);
             additionalInfo.put("license", "made by pigx");
			((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
			return accessToken;
		};
	}
}

```

2、其他认证：
```java
@Primary
@Order(90)
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
			.antMatchers("/actuator/**", "/oauth/removeToken").permitAll()
			.anyRequest().authenticated()
			.and().csrf().disable();
        //http.httpBasic();
	}

	@Bean
	@Override
	public AuthenticationManager authenticationManagerBean() throws Exception {
		return super.authenticationManagerBean();
	}

	@Bean
	public PasswordEncoder passwordEncoder() {
		return PasswordEncoderFactories.createDelegatingPasswordEncoder();
	}
}

```

3、其他（可选）
```java
/**
 * @author gqsu
 * @date 2018/6/24
 * 删除token端点
 */
@RestController
@AllArgsConstructor
public class RevokeTokenEndpoint {
	private final TokenStore tokenStore;

	/**
	 * 删除token
	 *
	 * @param authHeader Authorization
	 */
	@GetMapping("/oauth/removeToken")
	public R<Boolean> logout(@RequestHeader(value = "Authorization", required = false) String authHeader) {
		if (StringUtils.hasText(authHeader)) {
			String tokenValue = authHeader.replace("Bearer", "").trim();
			OAuth2AccessToken accessToken = tokenStore.readAccessToken(tokenValue);
			if (accessToken == null || StrUtil.isBlank(accessToken.getValue())) {
				return new R<>(false, "退出失败，token 为空");
			}
			tokenStore.removeAccessToken(accessToken);
		}
		return new R<>(Boolean.TRUE);
	}
}
```


