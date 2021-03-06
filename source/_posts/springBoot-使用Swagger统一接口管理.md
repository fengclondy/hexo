---
layout: post
title: springBoot使用swagger统一接口文档
date: 2018-02-25 12:53:30
tags: springBoot
categories: springBoot
---

### 集成swagger

1、引入依赖：
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.7.0</version>
</dependency>
```

2、创建Swagger2配置类，可单独新建包config或与application.java同级：
```java
@Configuration
@EnableSwagger2
public class Swagger2Config {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.njnsi.puzzleGame.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact("gqsu","http://njnsi.top","nsisgq@jit.deu.cn");

        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("南京软件研究院")
                .termsOfServiceUrl("http://njnsi.top")
                .contact(contact)
                .version("0.0.1")
                .build();
    }
}
```

<!-- more -->

3、添加文档内容（可选择性添加）：

1)在类上添加注解：

```java
@Api("swagger实例")
//@Apignore      //注解在类上，表示不让其在前端显示；注解在方法上，表示接口不再前端显示
```

2)在方法上添加注解：

```java
@ApiOperation(value="创建用户", notes="根据User对象创建用户")
@ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
//@ApiImplicitParams({
//            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long"),
//            @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
//    })
@RequestMapping(value="/", method=RequestMethod.POST)
   public String postUser(@ModelAttribute User user) {
       // 处理"/users/"的POST请求，用来创建User
       // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数
       users.put(user.getId(), user);
   return "success";
}
```

> @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "int", paramType = "path", example = "1000")

3)在实体类bean上添加注解：
```java
@ApiModel
public class User{

@ApiModelProperty(value="姓名"，example="")   //example是对象关联属性，不需要则不添加
private String name;
...
}
```

4、启动SpringBoot程序，查看文档：http://localhost:8080/swagger-ui.html

