---
layout: post
title: 分享的一些java代码片段
date: 2017-04-27 02:14:20
tags: share
categoriets: share
---


### 分享的一些代码

#### html代码转纯文本

```java
//将html转换为纯文本，此方法最后保留了&nbps空格，使用时注意将空格替换掉
public static String delHTMLTag(String htmlStr){ 
　　　String regEx_script="<script[^>]*?>[\\s\\S]*?<\\/script>"; //定义script的正则表达式 
　　　String regEx_style="<style[^>]*?>[\\s\\S]*?<\\/style>"; //定义style的正则表达式 
　　　String regEx_html="<[^>]+>"; //定义HTML标签的正则表达式 

　　　Pattern p_script=Pattern.compile(regEx_script,Pattern.CASE_INSENSITIVE); 
　　　Matcher m_script=p_script.matcher(htmlStr); 
　　　htmlStr=m_script.replaceAll(""); //过滤script标签 

　　　Pattern p_style=Pattern.compile(regEx_style,Pattern.CASE_INSENSITIVE); 
　　　Matcher m_style=p_style.matcher(htmlStr); 
　　　htmlStr=m_style.replaceAll(""); //过滤style标签 

　　　Pattern p_html=Pattern.compile(regEx_html,Pattern.CASE_INSENSITIVE); 
　　　Matcher m_html=p_html.matcher(htmlStr); 
　　　htmlStr=m_html.replaceAll(""); //过滤html标签

　　　return htmlStr.trim(); //返回文本字符串 
}
```

<!-- more -->