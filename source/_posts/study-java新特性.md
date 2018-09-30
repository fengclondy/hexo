---
layout: post
title: study-java新特性
date: 2018-08-02 02:17:35
tags: study
categories: study
---


### java8新特性
>参考:[文章](https://blog.csdn.net/op134972/article/details/76408237?locationNum=1&fps=1)

- Lambda 表达式 − Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中。
- 方法引用 − 方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。
- 默认方法 − 默认方法就是一个在接口里面定义了一个实现的方法。
- 新工具 − 新的编译工具，如：Nashorn引擎 jjs、 类依赖分析器jdeps。
- Stream API −新添加的Stream API（java.util.stream） 把真正的函数式编程风格引入到Java中。
- Date Time API − 加强对日期与时间的处理。
- Optional 类 − Optional 类已经成为 Java 8 类库的一部分，用来解决空指针异常。
- Nashorn, JavaScript 引擎 − Java 8提供了一个新的Nashorn javascript引擎，它允许我们在JVM上运行特定的javascript应用。
- Base64编码。
<!-- more -->

```java
// 使用 java 7 排序
private void sortUsingJava7(List<String> names){   
    Collections.sort(names, new Comparator<String>() {
        @Override
        public int compare(String s1, String s2) {
        return s1.compareTo(s2);
        }
    });
}
   
// 使用 java 8 排序
private void sortUsingJava8(List<String> names){
    Collections.sort(names, (s1, s2) -> s1.compareTo(s2));
}
```

```java
// 获取当前的日期时间 （本地）
LocalDateTime currentTime = LocalDateTime.now();
System.out.println("当前时间: " + currentTime);
LocalDate date1 = currentTime.toLocalDate();
// 获取当前时间日期 （时区）
ZonedDateTime date1 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
ZoneId id = ZoneId.of("Europe/Paris");
```

```shell
Optional类常用方法： 
- Optional.of(T t) : 创建一个Optional 实例 
- Optional.empty() : 创建一个空的Optional 实例 
- Optional.ofNullable(T t):若t 不为null,创建Optional 实例,否则创建空实例 
- isPresent() : 判断是否包含值 
- orElse(T t) : 如果调用对象包含值，返回该值，否则返回t 
- orElseGet(Supplier s) :如果调用对象包含值，返回该值，否则返回s 获取的值 
- map(Function f): 如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty() 
- flatMap(Function mapper):与map 类似，要求返回值必须是Optional
```

总的来说，JDK在以下方面具有新特性： 
1. 速度更快 – HashMap中使用红黑树 
2. 代码更少 – Lambda 
3. 强大的Stream API – Stream 
4. 便于并行 – Parallel 
5. 最大化减少空指针异常 – Optional

参考：https://mp.weixin.qq.com/s/yXkQwcKueWYLDKw3l8tFDQ



### java11新特性
- 1、本地变量类型推断
```java
var javastack = "javastack";
//等价于 String javastack = "javastack"; 即不需要写出具体的类型
```

- 2、字符串加强
```java
/** 增加了一系列的字符串处理方法 */
// 判断字符串是否为空白
" ".isBlank(); // true

// 去除首尾空格
" Javastack ".strip(); // "Javastack"

// 去除尾部空格 
" Javastack ".stripTrailing(); // " Javastack"

// 去除首部空格 
" Javastack ".stripLeading(); // "Javastack "

// 复制字符串
"Java".repeat(3); // "JavaJavaJava"

// 行数统计
"A\nB\nC".lines().count(); // 3
```

3、集合加强
Jdk 里面为集合（List/ Set/ Map）都添加了 `of` 和 `copyOf` 方法，它们两个都用来创建不可变的集合。

4、Stream 加强
1) 增加单个参数构造方法，可为null
```java
Stream.ofNullable(null).count(); // 0
```
2) 增加 takeWhile 和 dropWhile 方法
```java
Stream.of(1, 2, 3, 2, 1)
    .takeWhile(n -> n < 3)
    .collect(Collectors.toList());  // [1, 2]
```
从开始计算，当 n < 3 时就截止。

```java
Stream.of(1, 2, 3, 2, 1)
    .dropWhile(n -> n < 3)
    .collect(Collectors.toList());  // [3, 2, 1]
```
这个和上面的相反，一旦 n < 3 不成立就开始计算。

3）iterate重载

这个 iterate 方法的新重载方法，可以让你提供一个 Predicate (判断条件)来指定什么时候结束迭代。

如果你对 JDK 8 中的 Stream 还不熟悉，可以看之前分享的这一系列教程。

5、Optional 加强
```java
Optional.of("javastack").orElseThrow();     // javastack
Optional.of("javastack").stream().count();  // 1
Optional.ofNullable(null)
    .or(() -> Optional.of("javastack"))
    .get();   // javastack
```

6、InputStream 加强
```java
var classLoader = ClassLoader.getSystemClassLoader();
var inputStream = classLoader.getResourceAsStream("javastack.txt");
var javastack = File.createTempFile("javastack2", "txt");
try (var outputStream = new FileOutputStream(javastack)) {
    inputStream.transferTo(outputStream);
}
```
7、HTTP Client API
```java
var request = HttpRequest.newBuilder()
    .uri(URI.create("https://javastack.cn"))
    .GET()
    .build();
var client = HttpClient.newHttpClient();

// 同步
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());

// 异步
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println);
```