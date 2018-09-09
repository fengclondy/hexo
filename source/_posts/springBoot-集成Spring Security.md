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
```java
# security
security.user.name=admin
security.user.password=admin
```

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