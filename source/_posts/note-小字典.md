---
layout: post
title: 小字典
date: 2018-03-15 06:16:42
tags: note
categories: note
---

### 好用的开源库

#### Apache Commons Lang

```xml
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.4</version>
</dependency>
```

```java
if(StringUtils.isBlank(inputString)){...}
```

#### log4j

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.5</version>
</dependency>
```

#### <!-- more -->

#### Jackson

Jackson是一个多用途的Java库，用于处理JSON数据。使用它可以很方便地在JSON数据和Java对象之间进行转换。
```java
  ObjectMapper mapper = new ObjectMapper(); 
  User user = mapper.readValue(new File("user.json"), User.class);
```
#### Gson

Google开发的JSON库，可以实现JSON字符串与JAVA对象之间的转换，使用起来也非常方便。
```java
  Gson gson = new Gson();
  String[] strings = {"abc", "def", "ghi"};
  gson.toJson(strings);  // ==> ["abc", "def", "ghi"]
```

#### Lombok

#### JUnit

#### Apache POI

office文档处理工具,能处理word，excel，ppt等

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.14</version>
</dependency>
```


### 安装curl工具

[下载网址](https://curl.haxx.se/download.html) 

下载解压完成后，通过cmd进入src目录，运行命令： 

(1)显示http response的头信息: `curl -i www.baidu.com`  

(2)显示一次http请求的通信过程: `curl -v www.baidu.com` 或者输出为文档：`curl --trace output.txt www.baidu.com`  

(3)执行get/post/put/delete操作请求： `curl -x get/post/put/delete www.example.com`