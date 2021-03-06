---
layout: post
title: 基于Token的WEB后台认证机制
date: 2018-04-09 02:37:52
tags: note
categories: note
---

几种常用的认证机制

### HTTP Basic Auth

HTTP Basic Auth简单点说明就是每次请求API时都提供用户的username和password，简言之，Basic Auth是配合RESTful API 使用的最简单的认证方式，只需提供用户名密码即可，
但由于有把用户名密码暴露给第三方客户端的风险，在生产环境下被使用的越来越少。因此，在开发对外开放的RESTful API时，尽量避免采用HTTP Basic Auth


### OAuth

OAuth（开放授权）是一个开放的授权标准，允许用户让第三方应用访问该用户在某一web服务上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。

OAuth允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的第三方系统
（例如，视频编辑网站)在特定的时段（例如，接下来的2小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。
这样，OAuth让用户可以授权第三方网站访问他们存储在另外服务提供者的某些特定信息，而非所有内容

<!-- more -->

下面是OAuth2.0的流程：
![](http://p2jr3pegk.bkt.clouddn.com/jwt01.png)

这种基于OAuth的认证机制适用于个人消费者类的互联网产品，如社交类APP等应用，但是不太适合拥有自有认证权限管理的企业应用；


### Cookie Auth

Cookie认证机制就是为一次请求认证在服务端创建一个Session对象，同时在客户端的浏览器端创建了一个Cookie对象；
通过客户端带上来Cookie对象来与服务器端的session对象匹配来实现状态管理的。
默认的，当我们关闭浏览器的时候，cookie会被删除。但可以通过修改cookie 的expire time使cookie在一定时间内有效；


### Token Auth


![](http://p2jr3pegk.bkt.clouddn.com/jwt02.png)

Token Auth的优点:

Token机制相对于Cookie机制又有什么好处呢？

- 支持跨域访问: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输.
- 无状态(也称：服务端可扩展行):Token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息.
- 更适用CDN: 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片等），而你的服务端只要提供API即可.
- 去耦: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可.
- 更适用于移动应用: 当你的客户端是一个原生平台（iOS, Android，Windows 8等）时，Cookie是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。
- CSRF:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。
- 性能: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多.
- 不需要为登录页面做特殊处理: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做特殊处理.
- 基于标准化:你的API可以采用标准化的 JSON Web Token (JWT). 这个标准已经存在多个后端库（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）.


### 基于JWT的Token认证机制实现

JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。其

JWT的组成
一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。
载荷（Payload）

```
{ "iss": "Online JWT Builder", 
  "iat": 1416797419, 
  "exp": 1448333419, 
  "aud": "www.example.com", 
  "sub": "jrocket@example.com", 
  "GivenName": "Johnny", 
  "Surname": "Rocket", 
  "Email": "jrocket@example.com", 
  "Role": [ "Manager", "Project Administrator" ] 
}
```

- iss: 该JWT的签发者，是否使用是可选的；
- sub: 该JWT所面向的用户，是否使用是可选的；
- aud: 接收该JWT的一方，是否使用是可选的；
- exp(expires): 什么时候过期，这里是一个Unix时间戳，是否使用是可选的；
- iat(issued at): 在什么时候签发的(UNIX时间)，是否使用是可选的；
  其他还有：
- nbf (Not Before)：如果当前时间在nbf里的时间之前，则Token不被接受；一般都会留一些余地，比如几分钟；，是否使用是可选的；



### 认证过程

下面我们从一个实例来看如何运用JWT机制实现认证：

登录
第一次认证：第一次登录，用户从浏览器输入用户名/密码，提交后到服务器的登录处理的Action层（Login Action）；
Login Action调用认证服务进行用户名密码认证，如果认证通过，Login Action层调用用户信息服务获取用户信息（包括完整的用户信息及对应权限信息）；
返回用户信息后，Login Action从配置文件中获取Token签名生成的秘钥信息，进行Token的生成；
生成Token的过程中可以调用第三方的JWT Lib生成签名后的JWT数据；
完成JWT数据签名后，将其设置到COOKIE对象中，并重定向到首页，完成登录过程；

![](http://p2jr3pegk.bkt.clouddn.com/jwt03.png)

请求认证

基于Token的认证机制会在每一次请求中都带上完成签名的Token信息，这个Token信息可能在COOKIE
中，也可能在HTTP的Authorization头中；

客户端（APP客户端或浏览器）通过GET或POST请求访问资源（页面或调用API）；

认证服务作为一个Middleware HOOK 对请求进行拦截，首先在cookie中查找Token信息，如果没有找到，则在HTTP Authorization Head中查找；

如果找到Token信息，则根据配置文件中的签名加密秘钥，调用JWT Lib对Token信息进行解密和解码；

完成解码并验证签名通过后，对Token中的exp、nbf、aud等信息进行验证；

全部通过后，根据获取的用户的角色权限信息，进行对请求的资源的权限逻辑判断；

如果权限逻辑判断通过则通过Response对象返回；否则则返回HTTP 401；


对Token认证的五点认识

对Token认证机制有5点直接注意的地方：

一个Token就是一些信息的集合；

在Token中包含足够多的信息，以便在后续请求中减少查询数据库的几率；

服务端需要对cookie和HTTP Authrorization Header进行Token信息的检查；

基于上一点，你可以用一套token认证代码来面对浏览器类客户端和非浏览器类客户端；

因为token是被签名的，所以我们可以认为一个可以解码认证通过的token是由我们系统发放的，其中带的信息是合法有效的；


参考：http://www.cnblogs.com/xiekeli/p/5607107.html

https://blog.csdn.net/qq_28098067/article/details/52036493