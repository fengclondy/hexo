---
layout: post
title: Java多线程学习（详细）
date: 2018-05-02 05:45:59
tags: note
categories: note
---


### 一、进程与线程的区别

1、进程：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1--n个线程。（进程是资源分配的最小单位）

2、线程：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。（线程是cpu调度的最小单位）

3、区别：线程和进程一样分为五个阶段：创建、就绪、运行、阻塞、终止。
多进程是指操作系统能同时运行多个任务（程序）。
多线程是指在同一程序中有多个顺序流在执行。

在java中要想实现多线程，有两种手段，一种是继续Thread类，另外一种是实现Runable接口（其实准确来讲，应该有三种，还有一种是实现Callable接口，并与Future、线程池结合使用）

<!-- more -->

### 二、实现

#### 扩展java.lang.Thread类

```java
package com.multithread.learning;  
/** 
 *@functon 多线程学习 
 *@author George93 
 *@time 2017.6.7 
 */  
class Thread1 extends Thread{  
    private String name;  
    public Thread1(String name) {  
       this.name=name;  
    }  
    public void run() {  
        for (int i = 0; i < 5; i++) {  
            System.out.println(name + "运行  :  " + i);  
            try {  
                sleep((int) Math.random() * 10);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
         
    }  
}  
public class Main {  
  
    public static void main(String[] args) {  
        Thread1 mTh1=new Thread1("A");  
        Thread1 mTh2=new Thread1("B");  
        mTh1.start();  
        mTh2.start();  
  
    }  
  
}  
```

说明：
程序启动运行main时候，java虚拟机启动一个进程，主线程main在main()调用时候被创建。
随着调用MitiSay的两个对象的start方法，另外两个线程也启动了，这样，整个应用就在多线程下运行。

注意：start()方法的调用后并不是立即执行多线程代码，而是使得该线程变为可运行态（Runnable），什么时候运行是由操作系统决定的。
 Thread.sleep()方法调用目的是不让当前线程独自霸占该进程所获取的CPU资源，以留出一定时间给其他线程执行的机会。


#### 实现java.lang.Runnable接口

采用Runnable也是非常常见的一种，我们只需要重写run方法即可

```java
/** 
 *@functon 多线程学习 
 *@author George93 
 *@time 2017.6.7 
 */  
package com.multithread.runnable;  
class Thread2 implements Runnable{  
    private String name;  
  
    public Thread2(String name) {  
        this.name=name;  
    }  
  
    @Override  
    public void run() {  
          for (int i = 0; i < 5; i++) {  
                System.out.println(name + "运行  :  " + i);  
                try {  
                    Thread.sleep((int) Math.random() * 10);  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
          
    }  
      
}  
public class Main {  
  
    public static void main(String[] args) {  
        new Thread(new Thread2("C")).start();  
        new Thread(new Thread2("D")).start();  
    }  
  
}  
```

说明：
Thread2类通过实现Runnable接口，使得该类有了多线程类的特征。run()方法是多线程程序的一个约定。所有的多线程代码都在run方法里面。

#### Thread和Runnable的区别

>如果一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。

总结：

实现Runnable接口比继承Thread类所具有的优势：

- 1）：适合多个相同的程序代码的线程去处理同一个资源

- 2）：可以避免java中的单继承的限制

- 3）：增加程序的健壮性，代码可以被多个线程共享，代码和数据独立

- 4）：线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类

 

>补充：在java中，每次程序运行至少启动2个线程。一个是main线程，一个是垃圾收集线程。
因为每当使用java命令执行一个类的时候，实际上都会启动一个 JVM，每一个 JVM 实际就是在操作系统中启动了一个进程。
在java中所有的线程都是同时启动的，至于什么时候，哪个先执行，完全看谁先得到CPU的资源。