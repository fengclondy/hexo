---
layout: post
title: Spring boot 配置Servelt、Filter、Listener
date: 2018-03-16 08:13:58
tags: springBoot
categories: springBoot
---


。SpringBoot提供了2种方式配置Servlet、Listener、Filter。一种是基于RegistrationBean，另一种是基于注解。
### 1、基于RegistrationBean的配置

spring boot提供了 ServletRegistrationBean，FilterRegistrationBean，ServletListenerRegistrationBean这3个东西来进行配置Servlet、Filter、Listener

WebConfig.java
```JAVA
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter{
	@Bean
	public FilterRegistrationBean getDemoFilter(){
		DemoFilter demoFilter=new DemoFilter();
		FilterRegistrationBean registrationBean=new FilterRegistrationBean();
		registrationBean.setFilter(demoFilter);
		List<String> urlPatterns=new ArrayList<String>();
		urlPatterns.add("/*");//拦截路径，可以添加多个
		registrationBean.setUrlPatterns(urlPatterns);
		registrationBean.setOrder(1);
		return registrationBean;
	}
	@Bean
	public ServletRegistrationBean getDemoServlet(){
		DemoServlet demoServlet=new DemoServlet();
		ServletRegistrationBean registrationBean=new ServletRegistrationBean();
		registrationBean.setServlet(demoServlet);
		List<String> urlMappings=new ArrayList<String>();
		urlMappings.add("/demoservlet");////访问，可以添加多个
		registrationBean.setUrlMappings(urlMappings);
		registrationBean.setLoadOnStartup(1);
		return registrationBean;
	}
	@Bean
	public ServletListenerRegistrationBean<EventListener> getDemoListener(){
		ServletListenerRegistrationBean<EventListener> registrationBean
		                           =new ServletListenerRegistrationBean<>();
		registrationBean.setListener(new DemoListener());
//		registrationBean.setOrder(1);
		return registrationBean;
	}
}
```

参考：http://www.tianshouzhi.com/api/tutorials/springboot/106

<!-- more -->

### 2、给予注解的配置