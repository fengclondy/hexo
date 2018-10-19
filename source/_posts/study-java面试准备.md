---
layout: post
title: study--java面试
date: 2018-07-28 09:09:06
tags: study
categories: study
---

### java基本类型
八大基本类型：字符类型 char、布尔类型 boolean、整数类型 byte、short、int、long、
浮点数类型 float、double。

### 线程的两种实现方式和区别（准确来说是三种）

有两种实现方法，分别是继承 Thread 类与实现 Runnable 接口。 实现Runnable 接口除了拥有和继承 Thread类一样的功能以外，实现 Runnable 接口还具有以下功能。

- 适合多个相同程序代码的线程去处理同一资源的情况，可以把线程同程序中的数据有效的分离， 较好地体现了面向对象的设计思想。
- 可以避免由于 Java 的单继承特性带来的局限。例如，class Student 已经继承了 class Person，如 果要把 Student 类放入多线程中去，那么就不能使用继承 Thread 类的方式。
  因为在 Java 中规定 了一个类只能有一个父类，不能同时有两个父类。所以就只能使用实现 Runnable 接口的方式了。
- 增强了代码的健壮性，代码能够被多个线程共同访问，代码与数据是独立的。多个线程可以操 作相同的数据，与它们的代码无关。
  当线程被构造时，需要的代码和数据通过一个对象作为构 造函数实参传递进去，这个对象就是一个实现了 Runnable 接口的类的实例

<!-- more -->

### 线程同步的方法

同步是一种各线程间协调使用共享资源的一种方式。各线程间的相互通信是实现同步的重要因素， 所以 Java 提供了 wait()和 notify()等方法来使线程之间可以相互通信。
- wait()：使线程处于等待状态，并且释放所持有的对象的 lock。可以与 notify()方法配套使用。 它有两种形式，一种是以毫秒为单位的一段时间作为参数，另一种是没有参数。 
- sleep()：使一个正在运行的线程处于阻塞状态，可以以毫秒为单位的一段时间作为参数，它可 以使得线程在设定的时间停止运行，但是在设定的时间一过，线程重新进入可执行状态。由于 sleep()是一个静态方法，所以调用此方法要捕捉 InterruptedException 异常。 
- notify()：唤醒一个处于等待状态的线程，注意的是在调用此方法的时候，并不能确切的唤醒某 一个等待状态的线程，而是由 JVM 确定唤醒哪个线程，而且不是按优先级。 
- allnotity()：唤醒所有处入等待状态的线程，注意并不是给所有唤醒线程一个对象的锁，而是让 它们竞争。只有获得锁的那一个线程才能进入可执行状态。 


>sleep 是线程类（Thread）的方法，导致此线程暂停执行指定时间，给执行机会给其他线程，但是监 控状态依然保持，到时后会自动恢复。调用 sleep 不会释放对象锁。     
wait 是 Object 类的方法，对此对象调用 wait 方法导致本线程放弃对象锁，进入等待此对象的等待 锁定池。只有针对此对象发出 notify 方法（或 notifyAll）后，本线程才进入对象锁定池准备获得对象锁 进入运行状态。 


### Spring的核心主要有三点：

IoC：反转控制。
　　反转控制就是指将控制权由类内部抽离到容器，由容器类的实例化及动作进行配置管理。

Dependency-injection：依赖注入 
　　对象的依赖关系由负责协调系统中各个对象的第三方组件在创建对象时设定。对象不自行创建或管理它们的依赖关系，依赖关系被自动注入到需要它们的对象中。通过参数和配置能够体会出“注入”这个词在这里有多形象。依赖注入的最大好处就是松耦合。不需要再类内部去和特定的类进行绑定，而是将一些依赖关系以参数的形式注入到类内部。

Aspect Oriented Programming：面向切向编程
 　　在软件开发中，分布于应用中多处的功能被称为横切关注点。这些横切关注点往往和业务逻辑是相分离的，将这些横切关注点与业务逻辑相分离正式AOP要解决的。AOP编程能够让遍布在应用各处的功能分离出来形成可重用的组件。是高内聚低耦合的又一个体现，将通用实现模块与核心业务模块相分离。



### Integer类型的数值
```java
public static void main(String args){

    Integer f1=100, f2=100, f3=150, f4=150;
    System.out.println(f1==f2);
    System.out.println(f3==f4)
}
```
如果整型字面量的值在-128 到 127 之间，那么不会 new 新的 Integer 对象，而是直接引用常量池
中的 Integer 对象，所以上面的面试题中 f1==f2 的结果是 true，而 f3==f4 的结果是 false。 

### List,Map,Set

List接口有三个实现类（LinkedList：基于链表实现，链表内存是散乱的，每一个元素存储本身内存地址的同时还
存储下一个元素的地址。链表增删快，查找慢；ArrayList：基于数组实现，非线程安全的，效率高，便于索引，但不
便于插入删除；Vector：基于数组实现，线程安全的，效率低）。 

Map 接口有三个实现类（HashMap：基于 hash 表的 Map 接口实现，非线程安全，高效，支持 null 值和 null
键；HashTable：线程安全，低效，不支持null值和null键；LinkedHashMap：是 HashMap的一个子类，保存了
记录的插入顺序；SortMap接口：TreeMap，能够把它保存的记录根据键排序，默认是键值的升序排序）。 

Set 接口有两个实现类（HashSet：底层是由 HashMap 实现，不允许集合中有重复的值，使用该方式时需要重
写 equals()和 hashCode()方法；LinkedHashSet：继承与 HashSet，同时又基于 LinkedHashMap 来进行实现，底
层使用的是LinkedHashMp）

### AOP

AOP是一种编程范式，提供从另一个角度来考虑程序结构以完善面向对象编程（OOP）。
AOP为开发者提供了一种描述横切关注点的机制，并能够自动将横切关注点织入到面向对象的软件系统中，从而实现了横切关注点的模块化。
AOP能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任，例如事务处理、日志管理、权限控制等，封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。

### ConcurrentHashMap 处理并发

HashMap,Hashtable与ConcurrentHashMap都是实现的哈希表数据结构，在随机读取的时候效率很高。Hashtable实现同步是利用synchronized关键字进行锁定的，其是针对整张哈希表进行锁定的，即每次锁住整张表让线程独占，在线程安全的背后是巨大的浪费。ConcurrentHashMap和Hashtable主要区别就是围绕着 **锁的粒度** 进行区别以及如何区锁定。

ConcurrentHashMap  1.7和1.8版本的是有区别的，主要体现在：存放数据的 HashEntry 改为 Node，但作用都是相同的；分段锁技术换成了 CAS + synchronized 来保证并发安全性。

先说说HashMap。HashMap底层是基于 数组和链表 进行实现的。具体的可查看源码
```java
public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}
 
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    
    if (initialCapacity > MAXIMUM_CAPACITY)
       initialCapacity = MAXIMUM_CAPACITY;
    
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +loadFactor);

    this.loadFactor = loadFactor;
    threshold = initialCapacity;
    init();
}
```
给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，
当数量达到了 16 * 0.75 = 12 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。
因此通常建议能提前预估 HashMap 的大小最好，尽量的减少扩容带来的性能损耗。

>个人理解：HashMap是有数组（也有叫做桶）和链表进行实现的，其中的链表在HashMap定义为Entry，它有三个属性，key，value和next。
>当对一个元素进行存操作的时候，首先会利用hash算法对key进行运算求出，要存入数组中的index位置，如果当前位置没有其他元素则直接将该元素放入该位置。
>如果当前位置有值，则将该位置元素置为该元素，并将next节点指向原来的元素，所以数组中永远存放的是最后插入的元素。（这就很好的解决了hash冲突的问题，可以存放多个元素）
>当对一个key进行取操作的时候，首先会根据key计算出其对应的index（HashMap中同一个key的index位置永远是固定的，即使按2的幂次即扩容），
>然后在根据equals()方法找到对应的Entry，这也很好的解释了“为什么覆写equals()方法一定要覆写HashCode()方法”，因为hashcode相同，equals不一定相同；equals相同，hashcode必同。

**总结**：HashMap 基于 hashing 原理，我们通过 put ()和 get ()方法储存和获取对象。当我们将键值对传递给 put ()方法时，它调用键对象的 hashCode ()方法来计算 hashcode，让后找到 bucket 位置来储存值对象。当获取对象时，通过键对象的 equals ()方法找到正确的键值对，然后返回值对象。HashMap 使用 LinkedList 来解决碰撞问题，当发生碰撞了，对象将会储存在 LinkedList 的下一个节点中。 HashMap 在每个 LinkedList 节点中储存键值对对象。

HashMap的遍历方式：
```java
### 方式1（推荐）
Iterator<Map.Entry<String, Integer>> entryIterator = map.entrySet().iterator();
while (entryIterator.hasNext()) {
    Map.Entry<String, Integer> next = entryIterator.next();
    System.out.println("key=" + next.getKey() + " value=" + next.getValue());
}

### 方式2（不推荐，效率低，先取key在取value）
Iterator<String> iterator = map.keySet().iterator();
while (iterator.hasNext()){
    String key = iterator.next();
    System.out.println("key=" + key + " value=" + map.get(key));
}
```

ConcurrentHashMap 采用了分段锁技术，其中 Segment 继承于 ReentrantLock。
不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，
理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。
每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment，在整个过程都不需要加锁。



> 其他并发类：`ConcurrentLinkedQueue`,`CopyOnWriteArrayList`,`CopyOnWriteArraySet`,`LinkedBlockingQueue`,`ArrayBlockingQueue`,`LinkedBlockingQueue`,`DelayQueue`,`LinkedTransferQueue`,`SynchronousQueue`


### Java 中的设计模式&回收机制 

总体来说设计模式分为三大类： 

创建型模式，共五种：工厂方法模式（普通(类似于定义的接口)，多个，静态）、抽象工厂模式、单例模式（饿汉和懒汉）、建造者模式、原型模式。 

结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。 

行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模
式、状态模式、访问者模式、中介者模式、解释器模式。 


- 单例模式：懒汉式、饿汉式、双重校验锁、静态加载，内部类加载、枚举类加载。保证一个类仅有一个实例，并提供一个访问它的全局访问点。

- 代理模式：动态代理和静态代理，什么时候使用动态代理。

- 适配器模式：将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

- 装饰者模式：动态给类加功能。

- 观察者模式：有时被称作发布/订阅模式，观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态发生变化时，会通知所有观察者对象，使它们能够自动更新自己。

- 策略模式：定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。

- 外观模式：为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

- 命令模式：将一个请求封装成一个对象，从而使您可以用不同的请求对客户进行参数化。

- 创建者模式：将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。

- 抽象工厂模式：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。



### 单例设计模式之饿汉式与懒汉式（Singleton）

设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。

目的是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。

单例(Singleton)设计模式是Java中常用的设计模式之一。在Java应用中，单例对象能保证在一个JVM中，该对象只有一个实例存在。这样设计有如下好处：
- 有些类创建比较复杂，这样可以节省系统开销。
- 省去了new运算符，降低了系统内容的使用频率，减轻了GC的压力。
- 有些类只需要使用一个，这样就可以使用单例了。

```java
/**
 * 单例设计模式，饿汉式(一上来就new对象)
 */
public class Singleton {
	//1.私有化构造方法，不允许外边直接创建对象
	private Singleton(){
	}
	//2.声明唯一的实例,使用 private static 修饰
	private static Singleton instance = new Singleton();
	//3.提供一个用于获取实例的方法，使用 public static 修饰
	public static Singleton getInstance(){
		return instance;
	}
}



/**
 * 单例设计模式，懒汉式(用到的时候才new对象)
 */
public class Singleton2 {
	//1.私有化构造方法，不允许外边直接创建对象
	private Singleton2(){
	}
	//2.声明唯一的实例,使用 private static 修饰
	private static Singleton2 instance = null;
	//3.提供一个用于获取实例的方法，使用 public static 修饰
	public static Singleton2 getInstance(){
		if(instance == null)
			instance = new Singleton2();
		return instance;
	}
}
```

>懒汉模式的特点是加载类时比较慢，但运行时获取对象的速度比较快，线程安全。
饿汉模式的特点是加载类时比较快，但运行时获取对象的速度比较慢，线程不安全。




### cookie和session的区别

1、cookie数据存放在客户的浏览器上，session数据放在服务器上。

2、cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗，考虑到安全应当使用session。

3、session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用cookie。

4、单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

5、可以考虑将登陆信息等重要信息存放为session，其他信息如果需要保留，可以放在cookie中。


### 索引的作用及索引的数据结构

mysql索引的用途：

- 保持数据的完整性；

- 优化数据的访问性能

- 改进表的链接（join）操作

- 对结果进行排序

- 简化聚合数据操作

 索引的数据结构：B-、B+、R-、散列

 ### 索引的实现方式

 1、B+树

 2、散列索引

 3、位图索引

 ### 对象锁和类锁

 1. synchronized 加到 static 方法前面是给class 加锁，即类锁；而synchronized 加到非静态方法前面是给对象上锁。

 2. 如果多线程同时访问同一类的 类锁（synchronized 修饰的静态方法）以及对象锁（synchronized 修饰的非静态方法）这两个方法执行是异步的，原因：类锁和对象锁是2中不同的锁。

 3. 类锁对该类的所有对象都能起作用，而对象锁不能


### 什么是控制反转(IOC)?什么是依赖注入?

控制反转是应用于软件工程领域中的，在运行时被装配器对象来绑定耦合对象的一种编程技巧，对象之间耦合关系在编译时通常是未知的。在传统的编程方式中，业务逻辑的流程是由应用程序中的早已被设定好关联关系的对象来决定的。在使用控制反转的情况下，业务逻辑的流程是由对象关系图来决定的，该对象关系图由装配器负责实例化，这种实现方式还可以将对象之间的关联关系的定义抽象化。而绑定的过程是通过”依赖注入”实现的。

控制反转是一种以给予应用程序中目标组件更多控制为目的设计范式，并在我们的实际工作中起到了有效的作用。

依赖注入是在编译阶段尚未知所需的功能是来自哪个的类的情况下，将其他对象所依赖的功能对象实例化的模式。这就需要一种机制用来激活相应的组件以提供特定的功能，所以依赖注入是控制反转的基础。否则如果在组件不受框架控制的情况下，框架又怎么知道要创建哪个组件？

在Java中依然注入有以下三种实现方式：

- 构造器注入

- Setter方法注入

- 接口注入

### 五种经典的I/O模型

1. Blocking I/O（阻塞I/O）：

```java
 //代码1
//在Java中使用同步阻塞I/O实现文件的读取
public static void main(String[] args) throws IOException {
    FileInputStream fis = new FileInputStream(new File(PRO_FILE_PATH));
    Properties pro = new Properties();
    pro.load(fis);
    for (Object key : pro.keySet()) {
        System.out.println(key);
        System.out.println(pro.getProperty((String)key));
    }
}
```

2. Nonblocking I/O（非阻塞I/O）

3. I/O Multiplexing（I/O多路复用）

4. Signal-Driven I/O（信号驱动I/O）

5. Asynchronous I/O （异步I/O）

```java
 //代码2
//node环境下异步读取一个文件
const fs = require('fs')
const file='/Users/lk/Desktop/pro.properties'
fs.readFile(file,'utf-8', (err,data)=>{console.log(data)});
```

### cookie

cookie是保存在本地终端的数据。cookie由服务器生成，发送给浏览器，浏览器把cookie以kv形式保存到某个目录下的文本文件内，
下一次请求同一网站时会把该cookie发送给服务器。由于cookie是存在客户端上的，所以浏览器加入了一些限制确保cookie不会被恶意使用，
同时不会占据太多磁盘空间，所以每个域的cookie数量是有限的。

### token

token的意思是“令牌”，是用户身份的验证方式，最简单的token组成:uid(用户唯一的身份标识)、time(当前时间的时间戳)、
sign(签名，由token的前几位+盐以哈希算法压缩成一定长的十六进制字符串，可以防止恶意第三方拼接token请求服务器)
还可以把不变的参数也放进token，避免多次查库

token选值：

1. 用设备号/设备mac地址作为Token（推荐）

客户端：客户端在登录的时候获取设备的设备号/mac地址，并将其作为参数传递到服务端。

服务端：服务端接收到该参数后，便用一个变量来接收同时将其作为Token保存在数据库，并将该Token设置到session中，客户端每次请求的时候都要统一拦截，
并将客户端传递的token和服务器端session中的token进行对比，如果相同则放行，不同则拒绝。

分析：此刻客户端和服务器端就统一了一个唯一的标识Token，而且保证了每一个设备拥有了一个唯一的会话。该方法的缺点是客户端需要带设备号/mac地址作为参数传递，
而且服务器端还需要保存；优点是客户端不需重新登录，只要登录一次以后一直可以使用，至于超时的问题是有服务器这边来处理，如何处理？若服务器的Token超时后，
服务器只需将客户端传递的Token向数据库中查询，同时并赋值给变量Token，如此，Token的超时又重新计时。

2. 用session值作为Token

客户端：客户端只需携带用户名和密码登陆即可。

客户端：客户端接收到用户名和密码后并判断，如果正确了就将本地获取sessionID作为Token返回给客户端，客户端以后只需带上请求数据即可。

分析：这种方式使用的好处是方便，不用存储数据，但是缺点就是当session过期后，客户端必须重新登录才能进行访问数据。

区别：

1、cookie数据存放在客户的浏览器上，session数据放在服务器上。

2、cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗
   考虑到安全应当使用session。

3、session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能
   考虑到减轻服务器性能方面，应当使用COOKIE。

4、单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

5、所以个人建议：
   将登陆信息等重要信息存放为SESSION
   其他信息如果需要保留，可以放在COOKIE中


#### 基于JWT的Token认证机制实现

- 组成：一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。

- 认证过程：

1. 登录
第一次认证：
第一次登录，用户从浏览器输入用户名/密码，提交后到服务器的登录处理的Action层（Login Action）；

Login Action调用认证服务进行用户名密码认证，如果认证通过，Login Action层调用用户信息服务获取用户信息（包括完整的用户信息及对应权限信息）；
返回用户信息后，Login Action从配置文件中获取Token签名生成的秘钥信息，进行Token的生成；

生成Token的过程中可以调用第三方的JWT Lib生成签名后的JWT数据；

完成JWT数据签名后，将其设置到COOKIE对象中，并重定向到首页，完成登录过程；

![](http://p2jr3pegk.bkt.clouddn.com/jwt03.png)

请求认证

基于Token的认证机制会在每一次请求中都带上完成签名的Token信息，这个Token信息可能在COOKIE
中，也可能在HTTP的Authorization头中；

客户端（APP客户端或浏览器）通过GET或POST请求访问资源（页面或调用API）；

认证服务作为一个Middleware HOOK 对请求进行拦截，首先在cookie中查找Token信息，如果没有找到，则在HTTP Authorization Head中查找；

如果找到Token信息，则根据配置文件中的签名加密秘钥，调用JWT Lib对Token信息进行解密和解码；

完成解码并验证签名通过后，对Token中的exp、nbf、aud等信息进行验证；

全部通过后，根据获取的用户的角色权限信息，进行对请求的资源的权限逻辑判断；

如果权限逻辑判断通过则通过Response对象返回；否则则返回HTTP 401；

### Sprint cloud 和 Sprint boot区别

Spring Boot:

旨在简化创建产品级的Spring应用和服务，简化了配置文件，使用嵌入式web服务器，含有诸多开箱即用微服务功能，可以和spring cloud联合部署。

 Spring Cloud：

微服务工具包，为开发者提供了在分布式系统的配置管理、服务发现、断路器、智能路由、微代理、控制总线等开发工具包。

SpringCloud的**基础功能**：

- 服务治理： Spring  Cloud Eureka
- 客户端负载均衡： Spring Cloud Ribbon
- 服务容错保护： Spring  Cloud Hystrix  
- 声明式服务调用： Spring  Cloud Feign
- API网关服务：Spring Cloud Zuul
- 分布式配置中心： Spring Cloud Config

SpringCloud的高级功能：

- 消息总线： Spring  Cloud Bus
- 消息驱动的微服务： Spring Cloud Stream
- 分布式服务跟踪： Spring  Cloud Sleuth

### 设计模式6大原则
1. 单一职责原则(SRP) ：就一个类而言，应该仅有一个引起它变化的原因。 
2. 开放封闭原则(ASD) ：类、模块、函数等等等应该是可以拓展的，但是不可修改。
3. 里氏替换原则(LSP) ：所有引用基类（父类）的地方必须能透明地使用其子类的对象 。
4. 依赖倒置原则(DIP) ：高层模块不应该依赖低层模块，两个都应该依赖于抽象。抽象不应该依赖于细节，细节应该依赖于抽象。 
5. 迪米特原则(LOD) ：一个软件实体应当尽可能少地与其他实体发生相互作用。 
6. 接口隔离原则(ISP) ：一个类对另一个类的依赖应该建立在最小的接口上。 

### StringBuilder 与 StringBuffer 的区别以及StringBuilder 与 String 的区别
1. StringBuilder效率高，线程不安全，StringBuffer效率低，线程安全。

2. String是不可变字符串，StringBuilder是可变字符串。为什么有这样的差异，可以深入源码去解析，比如String类内的 priver final  char  value[] 等方法的原因。

3. 如果是简单的声明一个字符串没有后续过多的操作，使用 String，StringBuilder 均可，若后续对字符穿做频繁的添加，删除操作，或者是在循环当中动态的改变字符串的长度应该用 StringBuilder。使用 String 会产生多余的字符串，占用内存空间。

### 读写锁与独占锁

与传统锁不同的是读写锁的规则是可以共享读，但只能一个写，总结起来为：读读不互斥，读写互斥，写写互斥，而一般的独占锁是：读读互斥，读写互斥，写写互斥，而场景中往往读远远大于写，读写锁就是为了这种优化而创建出来的一种机制。

注意是*读远远大于写*，一般情况下独占锁的效率低来源于高并发下对临界区的激烈竞争导致线程上下文切换。因此当并发不是很高的情况下，读写锁由于需要额外维护读锁的状态，可能还不如独占锁的效率高。因此需要根据实际情况选择使用。

ReadWriteLock ---读写锁

ReentrantLock ---可重入锁