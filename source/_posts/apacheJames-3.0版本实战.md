---
layout: post
title: ApacheJames 3.0版本实战
date: 2018-02-01 02:42:43
tags: Apache James
categories: Apache James
---

#### 安装与配置

##### 安装

1、官网下载 apache james 3.0应用包，并进行解压
![](http://p2jr3pegk.bkt.clouddn.com/james03-1.png)

<!-- more -->

2、进入`/conf`目录，去掉所有的-template。如：sqlResources-template.xml更名为sqlResources.xml
![](http://p2jr3pegk.bkt.clouddn.com/james03-2.png)

3、修改james-database.properties
```properties
database.driverClassName=com.mysql.jdbc.Driver
database.url=jdbc:mysql://localhost:3306/email
database.username=root
database.password=root
vendorAdapter.database=MYSQL
openjpa.streaming=false
```

4、执行如下命令,查看能否正常启动
```powershell
$ run.bat
```
正常启动的话，会在数据库里生成表：  
![](http://p2jr3pegk.bkt.clouddn.com/james03-03.png)


如果不能正常启动，请查看端口是否有被占用
一般情况下，可能需要修改的文件有:
>dnsservice.xml（修改DNS）  
domainlist.xml(改成自己的域名)  
james-database.properties（修改DB信息，conf/lib目录下放相应的jar包）  
mailetcontainer.xml（修改postmaster）  
smtpserver.xml (限制邮件大小： 10240，限制10M； 注释掉： 127.0.0.0/8，否则外网邮件发不出去；
修改helloName)

5、由于开启了smtp认证，需要进行一些其他操作：  

打开mailetcontainer.xml，注释掉如下语句：
```xml
<!--<mailet match="RemoteAddrNotInNetwork=127.0.0.1" class="ToProcessor">
       <processor>relay-denied</processor>
       <notice>550 - Requested action not taken: relaying denied</notice>
</mailet>-->
```

开启Imap服务，imapserver.xml：
```xml
<lmtpserver enabled="true">
```

修改smtpserver.xml和pop3server.xml配置
```xml
<!-- 将自动检测关闭，指定为自己的。默认是注释掉的，即开启状态 -->
<helloName autodetect="false">test.com</helloName>
```

6、James运行/启动命令：
```powershell
$ james install
$ james start
```
如果启动或创建出现拒绝访问应该是权限的问题，需管理员运行cmd  
如果自建项目，集成参考：http://blog.csdn.net/afgasdg/article/details/6706512

##### 快速启动
可参见：[官方文档](http://james.apache.org/server/3/quick-start.html)

这里，需要注意的是，添加域名/用户：
```powershell
$ james-cli -h localhost -p 9999 adddomain test.com
$ james-cli -h localhost -p 9999 adduser test@test.com test
$ james-cli -h localhost -p 9999 adduser test2@test.com test2
```

参考：[配置文件详解](https://www.jianshu.com/p/59e91d424f70)

##### 对比2.0版本
1、3.0的配置文件不在是一个，而是多个。可以理解为将原先的config配置拆分成了多个子配置。
2、2.0启动需要后台启动服务，且服务不能关闭（命令行），3.0直接启动james start即可，而run.bat类似于控制台
3、自定义的驱动包存放路径 `\conf\lib\`下

百度文库：
```
james2.0版本能收发邮件，不能修改邮件状态（已读、已删除..）、不能添加自定义文件夹（只有一个收件箱）、
不能移动邮件到其他文件夹（这些问题本质上是james2.0版本不支持imap协议）；
james3.0-M2版本不存在上述问题，但是wulun2.0还是3.0都不支持oracle链接
```

##### 备注
```
database.properties     :  数据库属性配置（当以DB作为Repository的时候应用）      
dnsservice.xml            ：配置DNS   
domainlist.xml            ：配置域列表   
fetchmail.xml              ：取邮件   
imapserver.xml            ：IMAP协议服务配置   
jcr-repository.xml        ：配置Jackrabbit repository   
jmx.properties             ：配置JMX参数，用于监控                     
lmtpserver.xml            ：配置 IMTP协议服务   
log4j.properties           ：日志配置   
mailbox.xml                ：邮件箱配置   
mailetcontainer.xml     ：Maillet容器参数配置   
mailrepositorystore.xml   ：邮件repository配置（DB JCR MEM etc..）   
pop3server.xml           ：  配置POP3协议服务   
recipientrewritetable.xml ：暂不知是什么作用    
smtpserver.xml             ：配置SMTP协议服务   
sqlResources.xml         ：配置Repository为DB时候的表结构   
usersrepository23.xml    ：暂不知于usersrepository.xml区别   
usersrepository.xml        ：用户Repository配置  
```

##### Bug修复
1、修改了配置文件，则一定要先james stop，然后执行run.bat，接着在james start。  
最好的步骤是james remove --> run.bat --> james install --> james start

2、运行`run.bat`报错，原因是步骤2没有执行，这点在官网文档的里有详细的说明。

3、利用foxmail进行本地验证时，发现无法发送/接受邮件，请查看smtpserver.xml和pop3server.xml配置
```xml
<!-- 将自动检测关闭，指定为自己的。默认是注释掉的，即开启状态 -->
<helloName autodetect="false">test.com</helloName>
```
