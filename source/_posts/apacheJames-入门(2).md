---
layout: post
title: 搭建Apache James邮件服务(2)
date: 2018-01-20 13:10:57
tags: Apache James
categories: Apache James
---


### java代码实现邮件发送

`注：以下是2.3版本`

##### 原理分析

 我先解释一下大概的流程：
 当james 接收到一个smtp请求时首先会交给matcher来进行一系列过滤动作。
 然后由matcher交给相应的mailet来进行处理。
 James就相当于一个 matcher 和 mailet 的容器。就像tomcat之于servlet。
 下面，我们自定义 matcher 和 mailet

<!-- more -->

##### 发送器
```java
package com.yy.jamesstudy;   
  
import java.util.Date;   
import java.util.Properties;   
  
import javax.mail.Authenticator;   
import javax.mail.Message;   
import javax.mail.PasswordAuthentication;   
import javax.mail.Session;   
import javax.mail.Transport;   
import javax.mail.internet.InternetAddress;   
import javax.mail.internet.MimeMessage;   
  
public class Mail {   
    private String mailServer, From, To, mailSubject, MailContent;   
  
    private String username, password;   
  
    private Session mailSession;   
  
    private Properties prop;   
  
    private Message message;   
  
    // Authenticator auth;//认证   
    public Mail() {   
        // 设置邮件相关   
        username = "kakaxi";   
        password = "kakaxi";   
        mailServer = "localhost";   
        From = "kakaxi@localhost";   
        To = "mingren@localhost";   
        mailSubject = "Hello Scientist";   
        MailContent = "How are you today!";   
    }   
    public void send(){   
        EmailAuthenticator mailauth =    
new EmailAuthenticator(username, password);   
        // 设置邮件服务器   
        prop = System.getProperties();   
        prop.put("mail.smtp.auth", "true");   
        prop.put("mail.smtp.host", mailServer);   
        // 产生新的Session服务   
        mailSession = mailSession.getDefaultInstance(prop,   
                (Authenticator) mailauth);   
        message = new MimeMessage(mailSession);   
  
        try {   
        message.setFrom(new InternetAddress(From)); // 设置发件人   
        message.setRecipient(Message.RecipientType.TO,    
new InternetAddress(To));// 设置收件人   
        message.setSubject(mailSubject);// 设置主题   
        message.setContent(MailContent, "text/plain");// 设置内容   
        message.setSentDate(new Date());// 设置日期   
        Transport tran = mailSession.getTransport("smtp");   
        tran.connect(mailServer, username, password);   
        tran.send(message, message.getAllRecipients());   
        tran.close();   
        } catch (Exception e) {   
            e.printStackTrace();   
        }   
    }   
    public static void main(String[] args) {   
        Mail mail;   
        mail = new Mail();   
        System.out.println("sending......");   
        mail.send();   
        System.out.println("finished!");   
    }   
  
}   
  
class EmailAuthenticator extends Authenticator {   
    private String m_username = null;   
  
    private String m_userpass = null;   
  
    void setUsername(String username) {   
        m_username = username;   
    }   
  
    void setUserpass(String userpass) {   
        m_userpass = userpass;   
    }   
  
    public EmailAuthenticator(String username, String userpass) {   
        super();   
        setUsername(username);   
        setUserpass(userpass);   
    }   
  
    public PasswordAuthentication getPasswordAuthentication() {   
        return new PasswordAuthentication(m_username, m_userpass);   
    }   
} 
```


##### Matcher 的实现
```java
package com.yy.jamesstudy;   
  
import javax.mail.MessagingException;   
import org.apache.mailet.GenericRecipientMatcher;   
import org.apache.mailet.MailAddress;   
/**  
 * System   jamesstudy  
 * Package  com.yy.jamesstudy  
 *   
 * @author Yang Yang  
 * Created on 2007-9-14下午02:17:07  
 */  
public class BizMatcher extends GenericRecipientMatcher {     //GenericRecipientMatcher 是一个针对收件人进行过滤的matcher. 
  
    public boolean matchRecipient(MailAddress recipient) throws MessagingException {   
        //只接受邮件地址包含鸣人的邮件   
          if (recipient.getUser().indexOf("mingren")!=-1) {   
              return true;   
          }   
        return false;   
    }   
}
```

##### Mailet 的实现
```java
package com.yy.jamesstudy;   
  
import java.io.IOException;   
import javax.mail.MessagingException;   
import javax.mail.internet.MimeMessage;   
import org.apache.mailet.GenericMailet;   
import org.apache.mailet.Mail;   
import org.apache.mailet.MailAddress;   
/**  
 * System   jamesstudy  
 * Package  com.yy.jamesstudy  
 *   
 * @author  Yang Yang  
 * Created on 2007-9-14下午02:16:31  
 */  
public class BizMaillet extends GenericMailet {   //GenericMailet 是一个mailet的基本实现
  
    public void service(Mail mail)  {   
        MailAddress ma = mail.getSender();   
        System.out.println("sender:"+ma.toInternetAddress().toString());   
        try {   
            MimeMessage mm = mail.getMessage();   
            System.out.println("content:"+(String)mm.getContent().toString());   
        } catch (IOException e) {   
            e.printStackTrace();   
        } catch (MessagingException e) {   
            e.printStackTrace();   
        }   
    }   
}
```

编译
   我们把这两个java文件的class文件编译成一个名字为：jamesstudy.jar 的jar文件。
    
##### 发布－Matcher 和 Mailet以及config.xml

1.发布jar文件

我们把这个jar文件发布到项目根目录 \apps\james\SAR-INF\lib 下面
    
>注意：如果没有找到相关目录，则需要先启动一遍james,相关的文件夹才会被创建。还有一点需要特别说明：lib目录是通过我们手动创建的。

2.将Matcher 和 Mailet发布到config.xml中，config.xml在james\apps\james\SAR-INF\下
       
1）我们首先找到如下内容: 
```xml
<mailetpackages>  
  <mailetpackage>org.apache.james.transport.mailetsmailetpackage>  
  <mailetpackage>org.apache.james.transport.mailets.smimemailetpackage>  
<mailetpackages>  
<matcherpackages>  
  <matcherpackage>org.apache.james.transport.matchersmatcherpackage>  
  <matcherpackage>org.apache.james.transport.matchers.smimematcherpackage>  
<matcherpackages>
```

2）加入包声明:
```xml
<mailetpackages>  
  <mailetpackage>com.yy.jamesstudymailetpackage>  
  <mailetpackage>org.apache.james.transport.mailetsmailetpackage>  
  <mailetpackage>org.apache.james.transport.mailets.smimemailetpackage>  
<mailetpackages>  
  
<matcherpackages>  
  <matcherpackage>com.yy.jamesstudy matcherpackage>  
  <matcherpackage>org.apache.james.transport.matchersmatcherpackage>  
  <matcherpackage>org.apache.james.transport.matchers.smimematcherpackage>  
<matcherpackages>
```

3）加入 Matcher 和 Mailet的关联关系
找到 `<processor name="root"></processor>`元素在它的下面加入
xml 代码:

```xml
  <mailet match="BizMatcher" class="BizMaillet"/> 
```

Mailet元素代表了一个matcher和一个mailet的组合。Match属性：是指matcher的类名。而class 属性：是指mailet的类名。
最后别忘了，保存退出。并且重新启动james服务器。

测试－ 验证我们的mail应用程序:

我们主要通过mail类来测试我们的应用。还记得我们刚才写的那个mail类吗？！在那个类中我们初始化了相关的信息.
                   username = "kakaxi";
                   password = "kakaxi";
                   mailServer = "localhost";
                   From = "kakaxi@localhost";
                   To = "mingren@localhost";
                   mailSubject = "Hello Scientist";
                   MailContent = "How are you today!";
发件人是卡卡西，收件人是mingren.这两个用户我们在前面都已经创建完毕。我们用他们来测试james的邮件收发以及mailet api的应用。
根据需求假设我们发给james 服务器（这里是james的默认配置：localhost）的邮件的收件人是鸣人。那么我们就能通过matcher监测到这封邮件，并且调用相应的mailet来进行处理。由mailet打印出相应的邮件发送者和正文。运行Mail类后得到
Using PHOENIX_HOME:   C:\james
Using PHOENIX_TMPDIR: C:\james\temp
Using JAVA_HOME:      C:\j2sdk1.4.2_02
Phoenix 4.2

James Mail Server 2.3.1
Remote Manager Service started plain:4555
POP3 Service started plain:110
SMTP Service started plain:25
NNTP Service started plain:119
FetchMail Disabled


sender:kakaxi@localhost
content:How are you today!

总结

最终我们看到发送者和正文的信息。That’s all ! 大功告成。
其实james的功能是非常非常强大的，尤其是它的Mailet API能够帮助我们完成很多与邮件邮件有关的工作如过滤垃圾邮件。
用它我们甚至可以搭建我们自己的企业邮件服务器。
我们通过james接收到的邮件，然后用matcher找到我们想要处理的邮件，然后通过mailet做一些业务上的处理。这篇文章涵盖的面很小。如果大家有兴趣可以研究一下james项目。
Config.xml实际上是最重要的文件，如果你把它研究透彻了那么就就算把Mailet API研究透了。
         
参考：http://www.cnblogs.com/softidea/p/5348683.html