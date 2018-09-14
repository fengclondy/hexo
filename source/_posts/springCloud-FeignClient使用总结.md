---
layout: post
title: springCloud--FeignClient使用总结
date: 2018-05-21 06:27:03
tags: springCloud
categories: springCloud
---


### @FeignClient绑定的参数必须通过value来绑定，不然会报错

参考形式：

```
@FeignClient(value = "yl-user-admin",fallback = UserInfoFeignServiceImpl.class)
public interface UserInfoFeign {

      /**
     * 调用外部服务获取用户信息
     *
     * @param username
     * @return
     */
    @RequestMapping(value = "api/user/info",method = RequestMethod.GET)
    List getUserInfo(@RequestParam(value = "username") String username);
...
}
```

通过@fallback属性，这里的UserInfoFeignServiceImpl是接口UserInfoFeign的降级处理实现类，
在实现类头部加上`@Compent`注解

<!-- more -->


### FeignClient接口，不能使用@GetMapping 之类的简化组合注解
必须使用非简化形式 @RequestMapping(value = "/simple/{id}", method = RequestMethod.GET)

### FeignClient接口中，如果使用到@PathVariable ，必须指定其value


### FeignClient多参数的构造
对于复杂对象，采用的是POST方式提交，即使指明采用GET，请求也不会生效


### 如果需要自定义单个Feign配置，Feign的@Configuration 注解的类不能与@ComponentScan 的包重叠

如果包重叠，将会导致所有的Feign Client都会使用该配置。