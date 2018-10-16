---
layout: post
title: jvm
date: 2018-10-08 9:51:17
tags: jvm
categoriets: jvm
---




JVM提供了3种类加载器： BootstrapClassLoader、 ExtClassLoader、 AppClassLoader分别加载Java核心类库、扩展类库以及应用的类路径( CLASSPATH)下的类库。JVM通过双亲委派模型进行类的加载，我们也可以通过继承 java.lang.classloader实现自己的类加载器。