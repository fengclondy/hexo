---
layout: post
title: Web全栈开发的一些前端知识
date: 2018-04-27 02:13:35
tags: web
categories: web
---

### EL
EL主要用于查找作用域中的数据,然后对它们执行简单操作；它不是编程语言，甚至不是脚本编制语言。通常与 JSTL 标记一起作用，能用简单而又方便的符号来表示复杂的行为。

#### EL表达式的格式
用美元符号`$`定界,内容包括在花括号`{}`中;    例如: ${loginInfoBean.suser}

此外，可以将多个表达式与静态文本组合在一起以通过字符串并置来构造动态属性值;   例如:Hello {loginInfoBean.suser} ${loginInfoBean.spwd}

<!-- more -->

#### EL语法
EL表达式由标识符、存取器、文字和运算符组成。

- (1)标识符。用来标识存储在作用域中的数据对象。EL 有 11 个保留标识符，对应于 11个EL隐式对象。除了11隐式对象外,假定所有其它标识符都用来标识作用域的变量。

EL中的11个隐藏对象：
`pageContext` 实例对应于当前页面的处理
`pageScope` 与页面作用域属性的名称和值相关联的Map类
`requestScope` 与请求作用域属性的名称和值相关联的Map类
`sessionScope` 与会话作用域属性的名称和值相关联的Map类
`applicationScope` 与应用程序作用域属性的名称和值相关联的Map类
`param` 按名称存储请求参数的主要值的 Map 类
`paramValues` 将请求参数的所有值作为 String 数组存储的 Map 类
`Header` 按名称存储请求头主要值的 Map 类
`headerValues` 将请求头的所有值作为 String 数组存储的 Map 类
`cookie` 按名称存储请求附带的 cookie 的 Map 类
`initParam` 按名称存储 Web 应用程序上下文初始化参数的Map类

例:
```
${abc} 相当于<%=pageContext.findAttribute(“abc”)%>
${og_1} <%=pageContext.findAttribute(“og_1”)%>
```
…等等;就是说{}内的标识符除了11个保留字之外都表示作用域中的数据对应的名。

如：${requestScope}中的requestScope是11个EL隐式对象之一,它不再表示作用域中数据,而是表示request作用域;


- (2)EL存取器：存取器用来检索对象的特性或集合的元素。 通过 “[]” 或 “.” 符号获取相关数据。

例:
```
${userBean.suser} 或 ${userBean[“suser”]}  //获取输出bean中的suser属性值;
${mcType[“id”]}   //获取map中key为id对应的值;
```

- (3)EL运算符：允许对数据和文字进行组合以及比较。

类别：

1.算术运算符: `+、-、*、/（或 div）和 %（或 mod）`
2.关系运算符 : `==(或eq)、!=(或ne)、<(或lt)、>(或gt)、<=(或le) 和 >=(或ge)`
3.逻辑运算符: `&&(或 and)、||(或or)和 !(或 not)`
4.验证运算符: `empty`

>（验证运算符(empty):对于验证数据特别有用。empty 运算符采用单个表达式作为其变量（也即，${empty input}），并返回一个布尔值，该布尔值表示对表达式求值的结果是不是“空”值。求值结果为 null 的表达式被认为是空，即无元素的集合或数组。如果参数是对长度为零的 String 求值所得的结果，则 empty 运算符也将返回 true)


- (4)EL文字 ：表示固定的值 — 数字、字符、字符串、布尔型或空值。在 EL 表达式中，数字、字符串、布尔值和 null 都可以被指定为文字值。字符串可以用单引号或双引号定界。布尔值被指定为 true 和 false.

### JSTL
JSTL(JSP Standard Tag Library,JSP标准标签库)是一个不断完善的开放源代码的JSP标签库。用于简化JSP脚本和美化页面。

#### JSTL部署
在EE应用程序中部署JSTL有两种方式:
(1)已存在的工程上部署
将`jstl.jar`和`standard.jar`两个包考到现有工程 `WebRoot/WEB-INF/lib` 目录下
将相关的`.tld`文件考到现有工程`WebRoot/WEB-INF`目录下;

(2))通过eclipse部署
新建工程的时候直接部署，在JSP使用JSTL-core标签库，core在jsp中的使用。
在 web.xml 中添加
```
<jsp-config>
　　<taglib>
　　　　<taglib-uri>http://java.sun.com/jsp/jstl/core</taglib-uri>
　　　　<taglib-location>/WEB-INF/c.tld</taglib-location>
　　</taglib>
</jsp-config>
```
在jsp文件中添加:
```
<%@ taglib prefix=“c” uri=“http://java.sun.com/jsp/jstl/core” %>或<%@ taglib prefix="c" uri="/WEB-INF/c.tld" %>
使用<c:out value=“HelloWorld” />
```
#### 相关操作
#### Core的操作作用域变量标签
获取输出作用域中变量 :    ` <c:out > 属性: value [default] [escapeXml]`
定义作用域中变量 :         `<c:set > 属性: value var [scope]`
删除作用域中变量：     ` <c:remove > 属性: var [scope]`

#### Core的条件控制标签
单分支条件  :       `<c:if > 属性:test [var] [scope]`
多分支条件 ：   
```
<c:choose>
	<c:whe> 属性: test
 　  <c:otherwise>
```

#### Core的其它标签

输出转换成的URL :        `<c:url > 属性:value [context] [var] [scope]  和  <jsp:include >`
相似用于包含其它页面的内容:`<c:import >属性:url [context] [charEncoding] [var] [scope]`
重定向 : 
```
<c:redirect >属性: url [context]   与  
<c:url><c:import><c:redirect>
```
配合使用,用于传参
```
<c:param >属性: name value
```

#### Core的循环控制标签
实现简单循环 :    `  <c:forEach > var='item' begin='5' end='10' step='2‘ varStatus=‘’`
实现迭代(遍历) :     ` <c:forEach > items='' var='item‘ varStatus=‘’`

(属性varStatus和var相似设置一个作用域变量;只是varStatus作用域变量中存的是包括运行状态的对象;此对象包含如下属性 :  current index count first last begin end step

简单循环
```
<%@ page language="java" contentType="text/html; charset=GBK"%>
<%@ taglib prefix="c" uri="/WEB-INF/c.tld" %>
<html>
　　<head>
　　　　<meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
　　　　<title>testjstl1</title>
　　</head>
　　<body>
　　　　<c:forEach var="i" step="1" begin="1" end="100">
　　　　　　${i} <br>
　　　　</c:forEach>
　　</body>
</html>
```

循环迭代:
```
<%@ page language="java" contentType="text/html; charset=GBK"%>
<%@ taglib prefix="c" uri="/WEB-INF/c.tld" %>
<html>
　　<head>
　　　　<meta http-equiv="Content-Type" content="text/html;charset=gb2312" />
　　<title>testjstl1</title>
　　</head>
　　<body>
　　　　<c:forEach var="mcBean" items="${mcList}" varStatus="mcStatus">
　　　　　　当前遍历索引:${mcStatus.index} ; 商品名:${mcBean.sname} ; ....<br>
　　　　</c:forEach>
　　</body>
</html>
```

#### 在JSP使用JSTL-format标签库
在 web.xml 中添加:
```
<jsp-config>
　　<taglib>
　　　　<taglib-uri>http://java.sun.com/jstl/fmt</taglib-uri>
　　　　<taglib-location>/WEB-INF/fmt.tld</taglib-location>
　　</taglib>
</jsp-config>
```
在jsp文件中添加 : `<%@ taglib prefix="fmt" uri="http://java.sun.com/jstl/fmt" %>`
使用:`<fmt:formatDate value=“” pattern=“yyyy-MM-dd” />`

#### Format常用标签

(1)格式化输出日期: `<fmt:formatDate > value type var pattern`

type取值:
- `short`: 10/19/00 6:07 PM
- `medium`: Oct 19, 2000 6:07:01 PM
- `long`: October 19, 2000 6:07:01 PM MDT
- `full`: Thursday, October 19, 2000 6:07:01 PM MDT

例: `<fmt:formatDate value=“” pattern=“yyyy/MM/dd” />`
　　
(2)格式化输出数字: `<fmt:formatNumber> value var pattern`

例:`<fmt:formatNumber value=“” pattern=“###.##” />`

format实例:
```
<%@ page language="java" contentType="text/html; charset=GBK"%>
<%@ taglib prefix="fmt" uri="/WEB-INF/fmt.tld" %>
<html>
　　<head>
　　　　<meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
　　　　<title>testjstl1</title>
　　</head>
　　<body>
　　　　<jsp:useBean id="curDate" class="java.util.Date" scope="page"/>
　　　　<fmt:formatDate value="${curDate}" pattern="yyyy-MM-dd HH:mm:ss"/><br>
　　　　
　　　　<fmt:formatNumber value="10.32898432" pattern="#.##"/><br>
　　　　 <% request.setAttribute("var1",3.1415926); %>
　　　　<fmt:formatNumber value="${var1}" pattern="#.##"/><br>
　　</body>
</html>
```

### Json对象

#### Json和Map区别

Json是一种特殊的Map，只是Json中的键值用冒号隔开，而Map中的是等号隔开.

```
{"vessel"="999","voyage"="SGQ0","imono"="010203"}      //Map格式
{"vessel"："999","voyage"："SGQ0","imono"："010203"}     //Json格式
```

在easyui中，对于map格式的数组，一般用List<Map<String,Object>>格式存放数据，如果知道对应的数据的model，可以直接用List<E(实体)>。

例如：
```
[
  {"vessel"="999","voyage"="SGQ0","imoNo"="010203"},
  {"vessel"="789","voyage"="GY","imoNo"="010204"}，
  {...}
]
```
对于这样的一个数组，要区中里面的数据，只有将其付给list（一个逗号隔开一个List），在从list中取出每一个Map（一个括号对应一个Map）。在从Map中取出要的值。


例如：
```
[
  {"vessel"："999","voyage"："SGQ0","imoNo"："010203"},
  {"vessel"："789","voyage"："GY","imoNo"："010204"}，
  {...}
]
```
对于json格式的数据，则需要用JsonArray进行接收。
```
String str = "{...}";
JsonArray  jsonArray = new JsonArray(str);
JSONObject Object = JsonArray.getJSONObject(i); 
```
