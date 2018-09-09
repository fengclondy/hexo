---
layout: post
title: 易混淆的概念
date: 2018-01-24 07:05:21
tags: study
categories: study
---

### Java中常见类，方法和变量

（1）数组中没有length()方法，它只有length属性。字符串中有length()方法，集合中没有length()方法，用size()方法获取集合中元素的数量。
（2）字符串变量（str1, str2...）相加，先开辟空间，再相加。字符串常量(如"ab", "122"...)相加，首先在字符串常量池中找，判断有没有这个常量值， 则需要创建；有的话，则直接返回。字符串相加，推荐使用StringBuffer。
（3）编码与解码格式一样（默认为GBK格式，一个中文对应两个字节。**utf-8编码，一个汉字对应三个字节**）。
（4）静态代码块 ，构造代码块，构造方法的优先级:静态代码块>构造代码块>构造方法...
（5）StringBuffer和数组的区别：都属于容器类型的变量，数组只能存储一种类型的数据，并且长度是固定的，StringBuffer可以存储任意类型的元素。
（6）从键盘上输入字符：`Scanner scan = new Scanner(System.in); String str = scan.nextLine();`