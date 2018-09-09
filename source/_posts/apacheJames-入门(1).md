---
layout: post
title: 搭建Apache James邮件服务(1)
date: 2018-01-14 11:44:31
tags: Apache James
categories: Apache James
---


#### 邮件服务器

##### 什么是邮件服务器

邮件服务器是一种用来负责电子邮件收发管理的设备，而邮件服务就是责邮件的收信和发信功能，
其最主要有pop和smtp两个协议。
James 是一个企业级的邮件服务器，它完全实现了smtp 和 pops 以及nntp 协议。
同时，james服务器又是一个邮件应用程序平台。
James的核心是Mailet API,而james 服务其实是一个mailet的容器。
它可以让你非常容易的实现出很强大的邮件应用程序。
James开源项目被广泛的应用于与邮件有关的项目中。你可以通过它来搭建自己的邮件服务器。
我们可以利用Mailet API,编程接口来实现自己所需的业务。
James集成了Avalon 应用程序框架以及Phoenix Avalon 框架容器。
Phoenix为james 服务器提供了强大的支持。需要说明的是Avalon开源项目目前已经关闭。

总结：James是一款十分优秀的邮件服务器，具有性能稳定、扩展性好、可配置性强、响应速度快、源码公开等优点。
同时，由于James的后台管理不够方便、缺少必要的技术支持且主要用于1000用户量以内的邮件系统等原因，限制了james的高端企业级应用。

<!-- more -->

#### Apache james

`注：以下是2.3版本`

##### 环境搭建（windows）

1、下载地址： https://james.apache.org/download.cgi

2、解压后，在 `james-2.3.2.1\bin` 下找到 `run.bat` 运行。

3、在 `james-2.3.2.1\apps\james\SAR-INF\`目录下找到 `config.xml`

4、配置`config.xml`,修改：
```xml
<!-- 配置这里，将这个xml文件的localhost替换成你的域名(这里测试使用gqsu.top)-->
  <postmaster>Postmaster@gqsu.top</postmaster>
```

5、邮件数据存储选择，根据xml来看，提供的有3中数据存储方式，默认是是文件存储（不推荐）的：
- file:// 文件存储 
- db:// 数据库存储 
- dbfile:// 数据文件式存储

注：文件配置，需要先建立文件。数据库详细的配置可参见附录中给出的MySql配置。

6、其他配置 
找到 ``<administrator_accounts>``,将`!changeme!`改掉,等会登录使用的就是这个密码。
```xml
<administrator_accounts>
  <!-- CHECKME! -->
  	<!-- Change the default login/password. -->
  	<account login="root" password="!changeme!"/>
</administrator_accounts>
```

找到`<servernames>`,将两`true`改为`false`
```xml
<servernames autodetect="false" autodetectIP="false">
  	<!-- CONFIRM? -->
  	<servername>gqsu.top</servername>
</servernames>
```

找到下面的代码，注释掉
```xml 
<mailet match="InSpammerBlacklist=dnsbl.njabl.org."
  				 class="ToProcessor">
  	<processor> spam </processor>
  	<notice>550 Requested action not taken: rejected - see https://njabl.org/</notice>
</mailet>
```

7、配置 DNSServer,采用谷歌的DNS配置
```xml 
<servers>
  <server>8.8.8.8</server>         
  <server>8.8.4.4</server> 
</servers>
```

8、再次运行 run.bat即可。如果看到以下界面，说明安装成功。
![](http://p2jr3pegk.bkt.clouddn.com/james01-1.png)

9、客户端登录
```powershell  
telnet localhost 4555
#因为是本机服务，所以直接使用localhost连接邮件服务器
```

10、添加用户
```powershell 
adduser test test
#添加一个账号： test@gqsu.top ，密码：test的用户
```

##### 环境搭建（linux）

与windows类似，下载和解压的包别搞错就行
```powershell 
#解压命令
$ tar -zxvf james-binary-2.3.2.1.tar.gz
```

 启动：`./run.sh`,下面配置参照windows。

##### 功能测试

1、在添加一个用户test2(这里备注：如果连接数据库用户的创建会自动添加进后台表中，并对密码进行内部加密)。
```powershell
$ adduser test2 test2
```

2、利用Foxmail设置

![](http://p2jr3pegk.bkt.clouddn.com/james01-2.png)

##### 主要实现功能

功能就是：

- 1.局域网内的所有邮件收信和发信（普通邮件，Html邮件，附件邮件）；
- 2.外网的邮件发送（需要开通25 端口（默认是不允许的）；使用加密其他端口 smtp 发送邮件）

##### Bugx修复

1、报错一 ： java.io.IOException: 文件名、目录名或卷标语法不正确。

原因：这个报错的原因是用户信息，邮件收发产生的数据存储路径有误，默认使用的是文件存储。
修复：

a. 找到 所有的 destinationURL ,默认配置的是 'file://var/mail....'，就是使用文件存储，这里将其注释。如下图所示：

![](http://p2jr3pegk.bkt.clouddn.com/james01-3.png)

b.把数据库驱动添加到 /lib/ 目录下，我这里用的是Mysql,所以将 mysql-connector-java-5.1.39-bin.jar 添加到 lib 文件夹下。

c.在数据库中创建一个名为  mail 的数据库。当再次运行后James内部就会自动创建几个表。如下:

![](http://p2jr3pegk.bkt.clouddn.com/james01-4.png)

2、报错二 ：`om.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Unknown database 'mail'`

原因：没有创建数据库 mail ,创建即可解决。这是由这项配置决定的，需要确认填写无误，还有数据库驱动已加入其中即可避免。

```xml
<data-source name="maildb" class="org.apache.james.util.dbcp.JdbcDataSource">
  <driver>com.mysql.jdbc.Driver</driver>
  <dburl>jdbc:mysql://127.0.0.1/mail?autoReconnect=true</dburl>
  <user>root</user>
  <password>yourMysqlPwd</password>
  <max>20</max> 
</data-source>
```

##### 附录1

MySql配置：

1、配置 1。重要说明：注释掉默认的  file:// ,这个在文件中有好几处地方，修改方式雷同，都要注释掉默认的这个，再打开数据库存储方式的配置
```xml
  <!--
  <inboxRepository>
       <repository destinationURL="file://var/mail/inboxes/" type="MAIL"/>
  </inboxRepository>
  -->
        
  <!--使用数据库作为数据存储-->
   <inboxRepository>
      <repository destinationURL="db://maildb/inbox/" type="MAIL"/>
   </inboxRepository>
```

2、配置 2。将mysql的驱动包拷贝到 `james2.3.2\lib`目录下。

3、配置 3。找到 `<data-source>`节点 配置数据库连接。
```xml
 <data-source name="maildb" class="org.apache.james.util.dbcp.JdbcDataSource">
  	<driver>com.mysql.jdbc.Driver</driver>
  	<dburl>jdbc:mysql://127.0.0.1/mail?autoReconnect=true</dburl>
  	<user>username</user>
  	<password>password</password>
  	<max>20</max>
 </data-source>
```

4、配置 4。在Mysql数据库中添加一个空的数据库 mail ,数据库只能是mail,以后的数据都将会存储在这个数据库中


##### 附录2

我的配置文件： [下载](http://p2jr3pegk.bkt.clouddn.com/james-config.xml?attname=)

简单说明：

1、processor 配置处理请求模块，包括了 root，error，transport，spam，virus，local-address-error，

relay-denied（拒绝），bounces（退回）
```
当james 接收到一个smtp请求时首先会交给matcher来进行一系列过滤动作。然后由matcher交给相应的mailet来进行处理。
对于一个mailet模块来说，它的match属性指的是matcher类名，而class属性指的是mailet类名。
```

match属性：（过滤器）
```powershell
ALL ----全部
SMTPAuthSuccessful  ---- SMTP授权用户
```

class属性：（事件触发函数）
```powershell
ToProcessor ---
Bounce  ----拒绝（发件人）
NotifyPostmaster  ----通知管理员
LocalDelivery ---本地设备
RemoteDelivery  --远程设备
ToRepository ---仓库
SetMailAttribute ---属性
DSNBounce ---
```

2、其他配置：DNS设置、远程管理设置、pop3设置、连接池配置、邮件内容存储配置（数据库）、用户存储模块、数据库连接模块、连接管理模块
socket管理模块、线程管理模块



