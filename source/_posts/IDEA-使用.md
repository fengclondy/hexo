---
layout: post
title: IDEA使用遇到的问题
date: 2018-04-28 02:51:17
tags: IDEA
categoriets: IDEA
---

### IDEA一些操作

#### 激活
方法一：下载安装程序，安装完成后不要运行程序，此时把电脑时间改为2099年，修改完成后在运行程序，
当袒护激活时间的时候选择使用30天，等进入页面后，把页面关闭，然后再把时间调整回来，再次打开程序即完成了激活。

#### 安装lombok插件
settings-->plugins-->Lombok plugin。

lombok常用注解含义：
```
@NonNull    #非空检查，如果参数为空，则抛出一个空指针异常
@Cleanup    #代表的资源会被自动关闭，默认是调用资源的close()方法，在输入流中使用，可以不用手动关闭
@Getter/@Setter
@ToString
@EqualsAndHashCode(callSuper=true)       #覆写equals和hashCode，callSuper默认为false，表示不调用父类
@NoArgsConstructor/@RequiredArgsConstructor /@AllArgsConstructor  #无参/有参/全参构造方法
@Data        #3,4,5的组合形式
@Value
@SneakyThrows  #用在方法上，类似于进行了try/catch处理，使用@SneakyThrows(Exception.class)的形式指定抛出哪种异常
@Synchronized  #锁
@Log     #日志
```

<!-- more -->

关于日志功能，可参考其他日志注解，功能都是一样的：
```
@CommonsLog
private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);
@JBossLog
private static final org.jboss.logging.Logger log = org.jboss.logging.Logger.getLogger(LogExample.class);
@Log
private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());
@Log4j
private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);
@Log4j2
private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);
@Slf4j
private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
@XSlf4j
private static final org.slf4j.ext.XLogger log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);
```

#### Presentation View 全屏最大化视图
1、导航栏 **View** 视图(Alt+V)，弹出窗口后选择 **Enter Presentation Mode**,即可。
2、可使用 **CTRL+E** 弹出最近操作过的文件，或者 **CTRL+N** 和 **CTRL+SHIFT+N** 定位文件。
3、使用 Alt+V ,选择 **Exit Presentation Mode** 退出。


#### 在代码内部编写json字符串，使用工具完成转义
1、定义一个接受字符串，如：`String jsonString：""`。
2、鼠标定位到双引号的里面，使用 **alt+enter** ，弹出 **inject language** 视图，
并选中 **Inject language or reference**，按下 **enter** 键即完成了选中操作。
3、再次将鼠标停留在双引号中间，**alt+enter**，即可看到 **Edit JSON Fragment**，
选中回车即可在小窗口里像前端一样直接编写JSON语法了。
4、使用 **ctrl+F4** 退出编辑视图

#### 看不到完整的类的名称
1、使用 **alt+1** 将鼠标定位到 **project** 视图里。
2、然后使用 **ctrl+shift+左右箭头** 来移动分界线。

#### ctrl+shift+enter 自动为代码添加收尾结束符，一般指的是大括号。

#### ctrl+alt+左右箭头 可以用来显示屏幕的上下左右显示方式。

### 常见问题

#### Idea在导入有maven项目时，不能自动识别pom.xml，显示为普通橙色xml文件
解决方法：点击最右侧侧边栏，点击添加（蓝的的小加号），选择你导入项目的pom.xml文件

#### 针对springBoot文件夹的正确颜色显示
解决办法：右击项目，选择`Mark Directory as`更改为对应的文件资源

#### 局域网内其他人无法访问本地运行的项目（微服务）
解决办法：将项目运行地址设置为本地ip地址，同时关闭防火墙


#### maven项目依赖
举例，一个项目的依赖如下：
```
---- app-parent
             |-- pom.xml (pom)
             |
             |-- app-util
             |        |-- pom.xml (jar)
             |
             |-- app-dao
             |        |-- pom.xml (jar)
             |
             |-- app-service
             |        |-- pom.xml (jar)
             |
             |-- app-web
                      |-- pom.xml (war)
```
父类app-parent的pom.xml文件：
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.myorg.myapp</groupId>
	<artifactId>app-parent</artifactId>
	<packaging>pom</packaging>
	<version>1.0-SNAPSHOT</version>
	<modules>
		<module>app-util</module>  <!-1->
		<module>app-dao</module>  <!-2->
		<module>app-service</module>  <!-3->
		<module>app-web</module>  <!-4->
	</modules>
</project>
```
Maven的坐标GAV（groupId, artifactId, version）在这里进行配置，这些都是必须的。特殊的地方在于，这里的packaging为pom。
所有带有子模块的项目的packaging都为pom。packaging如果不进行配置，它的默认值是jar，代表Maven会将项目打成一个jar包。
另外一个重点在于，会根据子模块的相互依赖关系整理一个build顺序，然后依次build(1->2->3->4)。

子模块的pom.xml文件：
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<parent>
		<artifactId>app-parent</artifactId>
		<groupId>org.myorg.myapp</groupId>
		<version>1.0-SNAPSHOT</version>
	</parent>
	<modelVersion>4.0.0</modelVersion>
	<artifactId>app-util</artifactId>
	<dependencies>
		<dependency>
			<groupId>commons-lang</groupId>
			<artifactId>commons-lang</artifactId>
			<version>2.4</version>
		</dependency>
	</dependencies>
</project>
```
子模块从父模块内集成了所有东西。
>注意：app-web是一个war包，所以该工程必须包含一个目录src/main/webapp。并在这个目录下拥有web应用需要的文件，如/WEB-INF/web.xml。
如果没有web.xml，Maven进行mvn clean package时会报告build失败。

