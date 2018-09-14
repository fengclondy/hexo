---
layout: post
title: 杂技
date: 2018-02-05 04:22:25
tags: note
categories: note
---

>阿里云开放端口
https://jingyan.baidu.com/album/95c9d20d624d1eec4e756125.html?picindex=3

>腾讯云开放端口，暂未找到官方说明，但1025端口被官方禁用。

1、文件改后缀：

```powershell
mv *.jpg  *.png
```

2、文件改名，将一堆asdXXX.jpg的文件，更改为123XXX.png的文件：

```powershell
mv asd*.jpg  123*.png
```

windows下的命令为`ren`

3、cmd 下java -jar运行一个程序，中文乱码（零时生效）：
```powershell
$ CHCP 65001
```
```
65001  UTF-8
950   繁体中文
936   简体中文
437   美式英语
```

<!-- more -->

4、使用hexo，换了电脑怎么更新博客？

https://showcc.github.io/2017/06/08/Change%20the%20computer%20update%20the%20hexo%20blog/
https://showcc.github.io/2017/05/28/Hexo-spfk-LiveRe/


5、[Vue开源项目汇总](https://github.com/opendigg/awesome-github-vue)

6、[手摸手，带你用vue撸后台](https://juejin.im/post/59097cd7a22b9d0065fb61d2)

7、[微信小程序开发视频教程](http://wxopen.club/topic/582d4999745f85100cd13a65)

8、[Express入门](http://www.expressjs.com.cn/starter/faq.html)

参考：https://blog.csdn.net/qq_35038153/article/details/78430359



9、解决阿里云的tomcat启动慢的问题: java -Djava.security.egd=file:/dev/./urandom -jar pigx-gateway.jar。即在java -jar前面添加参数`-Djava.security.egd=file:/dev/./urandom`

10、`java -jar XXX.jar &`和`nohup java -jar XXX.jar &` 区别：第一种会在ssh窗口关闭后停止运行；nohub不挂断运行，默认输出到nohub.out文件中，可指定输出路径：`nohup java -jar XXX.jar >temp.txt &`，当然你可以通过`jobs`查看后台所有运行任务，通过`fg 编号`将对应任务调回至前台。

参考：https://www.2cto.com/kf/201806/751742.html


11、java多参数的设置：
```java
public AjaxResult getTest(@RequestBody JsonParam... param){
}
```
参考博客：https://blog.csdn.net/qq_41701956/article/details/80162936