---
layout: post
title: study-java8新特性
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
```
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

```
// 获取当前的日期时间 （本地）
LocalDateTime currentTime = LocalDateTime.now();
System.out.println("当前时间: " + currentTime);
LocalDate date1 = currentTime.toLocalDate();
// 获取当前时间日期 （时区）
ZonedDateTime date1 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
ZoneId id = ZoneId.of("Europe/Paris");
```

```
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


