---
layout: post
title: springBoot-整合MongoDB
date: 2018-09-04 08:46:27
tags: springBoot
categories: springBoot
---

### springBoot整合MongoDB

#### 1、添加依赖：
```pom
<dependency>
	<groupId>org.springframework.boot</groupId> 
	<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
​````

#### 2、配置文件中添加访问配置信息：
​```yml
spring: 
  data:
    mongodb:
      uri:  mongodb://root:123456@192.168.1.201:27017/students
```

#### 3、利用 MongoTemplate 操作 MongoDB

创建 Student 实体类:
```java
public class Student {

	private String id;
	private String stuName;
	private Integer age;
	
	public Student(){}

    public Student(String id, String stuName, Integer age) {
 		this.id = id;
		this.stuName =stuName;
		this.age = age;
	}
    //省略getter和setter方法
} 
```

创建 IStudentService 接口:
```java
public interface IStudentService {

	void addStudent(Student  stu);
}
```

创建实现类:
```java
@Service("studentService") 
public class StudentServiceImpl implements IStudentService {

	@Autowired
	private MongoTemplate  mongoTemplate;

	@Override
	public void addStudent(Student stu) {
   		mongoTemplate.save(stu);
	}
} 
```

创建 Controller 类，调用  Service 实现业务逻辑:
```java
@RestController 
@RequestMapping("/student") 
public classStuController {

  @Autowired
  privateIStudentService  studentService;

  @RequestMapping("/addStudent")
  public  String addStudent(String stuName,Integerage){

     String msg = "success";
     String id = UUID.randomUUID().toString();
     Student stu =  new Student(id,stuName,age);
     studentService.addStudent(stu);
     return  msg;
  }
}
```