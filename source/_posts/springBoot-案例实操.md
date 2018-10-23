---
layout: post
title: springBoot经典小例子
date: 2018-01-26 07:46:04
tags: springBoot
categories: springBoot
---

### 使用缓存

1、pom.xml中添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

2、增加@EnableCaching注解开启缓存功能

### 使用redis缓存

>可参见本博客 redis 的安装与使用

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

<!-- more -->

### 使用javaMail发送邮件

1、pom.xml中添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

2、application.properties中加入配置：

```properties
spring.mail.host=smtp.139.com
spring.mail.username=用户名
spring.mail.password=密码
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
#spring.mail.default-encoding=UTF-8
#mail.fromMail.addr=15951617064@139.com  //以谁来发送邮件
```

3、编写实现类mailService：

```java
@Component
public class MailServiceImpl implements MailService{

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private JavaMailSender mailSender;

    @Value("${mail.fromMail.addr}")
    private String from;

    @Override
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);

        try {
            mailSender.send(message);
            logger.info("简单邮件已经发送。");
        } catch (Exception e) {
            logger.error("发送简单邮件时发生异常！", e);
        }
    }
}
```
>@Value()也可以通过@ConfigurationProperties(prefix = "mail.fromMail")引入固定的头，
但是后面的属性必须与定义的一致，可不写，一般在实体类中这样写的多。

4、通过单元测试来实现一封简单邮件的发送：

```java

@RunWith(SpringRunner.class)
@SpringBootTest
public class MailServiceTest {

    @Autowired
    private MailService MailService;

    @Test
    public void testSimpleMail() throws Exception {
        MailService.sendSimpleMail("597009281@qq.com","简单邮件测试","这是一份测试邮件，请忽略该内容");
    }
}
```

5、实现复杂邮件的发送： 

* (1)发送html元素：

```java
public void sendHtmlMail(String to, String subject, String content) {
    MimeMessage message = mailSender.createMimeMessage();

    try {
        //true表示需要创建一个multipart message
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        mailSender.send(message);
        logger.info("html邮件发送成功");
    } catch (MessagingException e) {
        logger.error("发送html邮件时发生异常！", e);
    }
}
```
```java
@Test
public void testHtmlMail() throws Exception {
    String content="<html>\n" +
            "<body>\n" +
            "    <h3>hello world ! 这是一封Html邮件!</h3>\n" +
            "</body>\n" +
            "</html>";
    MailService.sendSimpleMail("597009281@qq.com","html邮件测试",content);
}
```

* (2)发送图片等附件：

```java
public void sendAttachmentsMail(String to, String subject, String content, String filePath){
    MimeMessage message = mailSender.createMimeMessage();

    try {
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        FileSystemResource file = new FileSystemResource(new File(filePath));
        String fileName = filePath.substring(filePath.lastIndexOf(File.separator));
        helper.addAttachment(fileName, file);
        //helper.addAttachment("附件-2.jpg", file);
        
        mailSender.send(message);
        logger.info("带附件的邮件已经发送。");
    } catch (MessagingException e) {
        logger.error("发送带附件的邮件时发生异常！", e);
    }
}
```
```java
@Test
public void sendAttachmentsMail() {
    String filePath="e:\\tmp\\application.log";
    mailService.sendAttachmentsMail("ityouknow@126.com", "主题：带附件的邮件", "有附件，请查收！", filePath);
}
```

>添加多个附件可以使用多条`helper.addAttachment(fileName, file)`

* (3)发送带静态资源的邮件：

```java
public void sendInlineResourceMail(String to, String subject, String content, String rscPath, String rscId){
    MimeMessage message = mailSender.createMimeMessage();

    try {
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        FileSystemResource res = new FileSystemResource(new File(rscPath));
        helper.addInline(rscId, res);

        mailSender.send(message);
        logger.info("嵌入静态资源的邮件已经发送。");
    } catch (MessagingException e) {
        logger.error("发送嵌入静态资源的邮件时发生异常！", e);
    }
}
```
```java
@Test
public void sendInlineResourceMail() {
    String rscId = "neo006";
    String content="<html><body>这是有图片的邮件：<img src=\'cid:" + rscId + "\' ></body></html>";
    String imgPath = "C:\\Users\\summer\\Pictures\\favicon.png";

    mailService.sendInlineResourceMail("ityouknow@126.com", "主题：这是有图片的邮件", content, imgPath, rscId);
}
```

>添加多个图片可以使用多条`<img src='cid:" + rscId + "' >` 和 `helper.addInline(rscId, res)`来实现

* (4)发送模板邮件：

方式1（推荐）：

pom中导入thymeleaf的包：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

在resorces/templates下创建emailTemplate.html：

```java
<!DOCTYPE html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8"/>
        <title>Title</title>
    </head>
    <body>
        您好,这是验证邮件,请点击下面的链接完成验证,<br/>
        <a href="#" th:href="@{ http://www.ityouknow.com/neo/{id}(id=${id}) }">激活账号</a>
    </body>
</html>
```

解析模板并发送：

```java
@Test
public void sendTemplateMail() {
    //创建邮件正文
    Context context = new Context();
    context.setVariable("id", "006");
    String emailContent = templateEngine.process("emailTemplate", context);

    mailService.sendHtmlMail("ityouknow@126.com","主题：这是模板邮件",emailContent);
}
```

其他(自行体会)：

引入freemarker依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```
在resources/templates/下，创建一个模板页面mail.ftl（或者mail.html）
```html
<html>
<body>
    <h3>你好， ${username}, 这是一封模板邮件!</h3>
</body>
</html>
```

编写测试用例：

```java
@Autowired
private FreeMarkerConfigurer freeMarkerConfigurer; 

@Test
public void sendTemplateMail(){
    MimeMessage message = null;
    try {
        message = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
	    helper.setFrom("15951617064@139.com");
		helper.setTo("597009281@qq.com");
        helper.setSubject("主题：模板邮件");

        Map<String, Object> model = new HashMap();
        model.put("username", "gqsu");
        //修改 application.properties 文件中的读取路径，默认是resources下的template
        //FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        //configurer.setTemplateLoaderPath("classpath:templates");
        //读取 html 模板
        Template template = freeMarkerConfigurer.getConfiguration().getTemplate("mail.ftl");
        String html = FreeMarkerTemplateUtils.processTemplateIntoString(template, model);
        helper.setText(html, true);
     } catch (Exception e) {
         e.printStackTrace();
    }
      mailSender.send(message);
}
```


#### 上传图片

https://blog.csdn.net/wqh8522/article/details/78971260

#### 消息会话
springBoot工程中，添加依赖：
```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
如果是SpringCloud，改为：
```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```
>实际使用时一定要分清楚，只是普通的springBoot工程还是SpringCLoud工程