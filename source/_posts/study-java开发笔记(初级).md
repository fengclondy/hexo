---
layout: post
title: java开发笔记（初级）
date: 2018-01-24 07:05:21
tags: study
categories: study
---

#### java中时间Date类型的数据

包java.util.Date如果不加处理，直接从数据库中返回，返回的是YYYY-MM-DD格式

包java.sql.Date如果不加处理，返回的是纯数字，如：1516777252000

关于1516777252000格式的处理:

* 1.在定义的实体类中，更改返回的get方法为String，set不需要改。（主要是在oracle中使用）

形如：

```java
public String getCreateTime() {
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		String str = sdf.format(this.createTime);
		return str;
	}
```

* 2.在底层sql语句返回处，调用数据库方法。Mysql对应DateFormate，Oracel对应To_Date();


* 3、springboot的公共配置：http://blog.720ui.com/2016/springboot_appendix_common_application_properties/

<!-- more -->

#### 查看session下都存放了哪些值，并打印下来。

```
HttpSession session = request..getSession();

for(Enumeration e = session.getAttributeNames(); e.hasMoreElements();){
    System.out.println(e.nextElement());
}
```

#### select框的选择

```
$("#id").val()   //获取select option的value值

$("#id option:selected").text()    //获取选中的option的文本信息
```

#### 县及县以上行政区划代码划分，转换成json格式

数据源：http://www.stats.gov.cn/tjsj/tjbz/xzqhdm/

步骤：
1.复制数据源（原始数据源是根据空格来划分的）。查找替换，将^(.*)$替换为'$1'，并给其设定一个var数组array。

2.新建.html页面，拷贝刚才的数据源，同时加入如下script代码。并运行代码，接着在控制台就能输出想要的结果。    

```js
//记录结果
var areas = [];                                                                                        
//堆栈记录层级信息
var stack = [];
//前一个区域和父节点
var prev, parent;
//缩进位置
var pos = 0;
for(var i = 0; i < array.length; i++){
    //正则匹配缩进的空格、代码、名称，[^\x00-\xff]+可以匹配中文
    var groups = /^(\s*)(\d+?)\s*([^\x00-\xff]+)$/g.exec(array[i]);
    var start = groups[1].length;
    var area = {code: groups[2], name: groups[3]};
    if(start == 0){
        //没有缩进时
        stack = [];
        parent = areas;
    } else if(start > pos){
        //如果有缩进，就使用上一个区域作为上级区域
        stack.push(parent);
        prev.children = [];
        parent = prev.children;
    } else if(start < pos){
        //取出上级区域
        parent = stack.pop();
    }
    parent.push(area);
    pos = start;
    prev = area;
}
console.log(JSON.stringify(areas,null,4));
```

输出格式：
```json
[ { "code": "110000", "name": "北京市",

　　　　　　　　　　"children": [ { "code": "110100", "name": "市辖区",

　　　　　　　　　　　　　　　　　　"children": [ { "code": "110101", "name": "东城区" },....
```

也可以直接获取到所有的省份、城市和地区的单独json格式串，代码见下：

```js
//记录结果
var prov = [];
var city = [];
var county = [];

//第一个后继和第二个后继
var next, next2;

for(var i = 0; i < array.length; i++){
    //正则匹配缩进的空格、代码、名称，[^\x00-\xff]+可以匹配中文
    var groups = /^(\s*)(\d+?)\s*([^\x00-\xff]+)$/g.exec(array[i]);
    var start = groups[1].length;
    var area1 = {id: groups[2], prov: groups[3]};
    var area2 = {id: groups[2], city: groups[3], parent:''};
    var area3 = {id: groups[2], county: groups[3], parent:''};

    if(start == 0){
        //没有缩进时
        prov.push(area1);
        next = area1.id;
    }if(start == 1){
        //缩进为1
        area2.parent = next;
        city.push(area2);
        next2 = area2.id;
    }if(start == 2){
        //缩进为2
        area2.parent = next2;
        county.push(area2);
    }    
    
}
console.log(JSON.stringify(prov,null,4));
console.log(JSON.stringify(city,null,4));
console.log(JSON.stringify(county,null,4)); 
```

输出结果：
```json
var provinceJson = [{"id":"110000","province":"北京市"},...
var provinceJson = [{"id":"110100","city":"市辖区","parent":"110000"},...
var provinceJson = [{"id":"110101","county":"东城区","parent":"110100"},...
```


#### finally不一定必须执行，return在catch/finally中处理情况（建议亲自操刀试一下）

#### 如何彻底修改Eclipse中项目的名称

    一般来说，只需要一步即可：右键工程：Refactor->Rename，或选中工程按F2，修改名称。

#### MD5工具实现编码

```java
import java.security.MessageDigest;  
import java.security.NoSuchAlgorithmException;  
  
public class MD5Tool {  
  
    public static String md5(String string) {  
        byte[] hash;  
        try {  
            hash = MessageDigest.getInstance("MD5").digest(string.getBytes("UTF-8"));  
        } catch (NoSuchAlgorithmException e) {  
            throw new RuntimeException("Huh, MD5 should be supported?", e);  
        } catch (UnsupportedEncodingException e) {  
            throw new RuntimeException("Huh, UTF-8 should be supported?", e);  
        }  
  
        StringBuilder hex = new StringBuilder(hash.length * 2);  
        for (byte b : hash) {  
            if ((b & 0xFF) < 0x10) hex.append("0");  
            hex.append(Integer.toHexString(b & 0xFF));  
        }  
        return hex.toString();  
    }  
}
```

#### ArrayList集合中删除某个元素用Iterator进行操作

```java
Iterator iterator = answerList.iterator();
  while(iterator.hasNext()){
    answerList.remove(iterator.next());
}
```

#### jQuery.inArray()

描述：在数组中查找指定值并返回它的索引（如果没有找到，则返回-1）。

作用：也可以用来获取不重复的元素 。

```java
function GetUnique(inputArray)
{
    var outputArray = [];
     
    for (var i = 0; i < inputArray.length; i++)
    {
        if ((jQuery.inArray(inputArray[i], outputArray)) == -1)
        {
            outputArray.push(inputArray[i]);
        }
    }
    
    return outputArray;
}
```
#### java相关

1、ceil():大于等于x，并且与它最近的整数

   floor():小于等于x，并且和x最接近的整数

2、try...catch...finally。finally语句会先于return和throw语句执行，当执行完成后再return给调用处。

3、java对象初始化过程：

- (1)初始化父类中的静态成员变量和静态代码块。

- (2)初始化子类中的静态成员变量和静态代码块。

- (3)初始化父类中的普通成员变量和代码块，在执行父类的构造方法。

- (4)初始化子类的普通成员变量和代码块，在执行子类的构造方法。

4、==判断对象是否是同一个引用，判断字符相等用equals方法。

5、访问修饰符权限：private < fiendly（无修饰）< protected < public。

6、s为null，则s.length()会抛空指针异常。

7、运行态失去CPU资源变成就绪态，就绪态得到CPU资源变成运行态，运行态等待或者I/O请求变成阻塞态，阻塞态等待的事情发生就进入了就绪态。

8、全局函数或者属性的对象是windows。

9、sleep和wait的区别有：

- 1. 这两个方法来自不同的类分别是Thread和Object

- 2. 最主要是sleep方法没有释放锁，而wait方法释放了锁，使得敏感词线程可以使用同步控制块或者方法。

- 3. wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用 synchronized(x){ x.notify() //或者wait() }

- 4. sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常

10、java 中String是 immutable的，也就是不可变，一旦初始化，其引用指向的内容是不可变的。　

　也就是说，String str = “aa”；str=“bb”；第二句不是改变“aa”所存储地址的内容，而是另外开辟了一个空间用来存储“bb”；同时由str指向原来的“aa”，
　现在已经不可达，GC时会自动回收。str还是aa

11、java的垃圾收集机制主要针对新生代和老年代的内存进行回收，不同的垃圾收集算法针对不同的区域。所以java的垃圾收集算法使用的是分代回收。
一般java的对象首先进入新生代的Eden区域，当进行GC的时候会回收新生代的区域，新生代一般采用复制收集算法，将活着的对象复制到survivor区域中，
如果survivor区域装在不下，就查看老年代是否有足够的空间装下新生代中的对象，如果能装下就装下，否则老年代就执行FULL GC回收自己，老年代还是装不下，就会抛出OUtOfMemory的异常

12、Java中静态变量只能在类主体中定义，不能在方法中定义。 静态变量属于类所有而不属于方法。

13、定义在类中的变量是类的成员变量，可以不进行初始化，Java会自动进行初始化，如果是引用类型默认初始化为null,如果是基本类型例如int则会默认初始化为0
　　 局部变量是定义在方法中的变量，必须要进行初始化，否则不同通过编译
　　 被static关键字修饰的变量是静态的，静态变量随着类的加载而加载，所以也被称为类变量
　　 被final修饰的变量是常量

14、方法执行完毕，局部变量（即垃圾,没有堆内存引用的栈内存）消除。

15、SpringMVC的原理:
​    SpringMVC是Spring中的模块，它实现了mvc设计模式的web框架，首先用户发出请求，请求到达SpringMVC的前端控制器（DispatcherServlet）,
​    前端控制器根据用户的url请求处理器映射器查找匹配该url的handler，并返回一个执行链，前端控制器再请求处理器适配器调用相应的handler进行处理并返回给前端控制器一个modelAndView，
​    前端控制器再请求视图解析器对返回的逻辑视图进行解析，最后前端控制器将返回的视图进行渲染并把数据装入到request域，返回给用户。
　　DispatcherServlet作为springMVC的前端控制器，负责接收用户的请求并根据用户的请求返回相应的视图给用户。

16、构造方法是一种特殊的方法，具有以下特点。
　　（1）构造方法的方法名必须与类名相同。
　　（2）构造方法没有返回类型，也不能定义为void，在方法名前面不声明方法类型。
　　（3）构造方法的主要作用是完成对象的初始化工作，它能够把定义对象时的参数传给对象的域。
　　（4）一个类可以定义多个构造方法，如果在定义类时没有定义构造方法，则编译系统会自动插入一个无参数的默认构造器，这个构造器不执行任何代码。
　　（5）构造方法可以重载，以参数的个数，类型，顺序。

17、ArrayList是基于数组实现的，所以查询快，增删慢；LinkedList是基于链表实现的，所以查找慢，增删快。

18、forward和redirect区别：

- 1.从地址栏显示来说：
  forward --是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,所以它的地址栏还是原来的地址。
  redirect --是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL。
- 2.从数据共享来说：
  forward --转发页面和转发到的页面可以共享request里面的数据。
  redirect --不能共享数据。
- 3.从运用地方来说：
  forward --一般用于用户登陆的时候,根据角色转发到相应的模块。
  redirect --一般用于用户注销登陆时返回主页面和跳转到其它的网站等。
- 4.从效率来说
　　forward --高。
　　redirect --低。

19、类的加载过程  

JVM将类加载过程分为三个步骤：装载（Load），链接（Link）和初始化(Initialize)链接又分为三个步骤

- 1) 装载：查找并加载类的二进制数据；

- 2)链接：   1.验证：确保被加载类的正确性；  2.准备：为类的静态变量分配内存，并将其初始化为默认值；   3.解析：把类中的符号引用转换为直接引用；

- 3)初始化：为类的静态变量赋予正确的初始值；

20、 类的初始化（类什么时候才被初始化）

- 1）创建类的实例，也就是new一个对象

- 2）访问某个类或接口的静态变量，或者对该静态变量赋值

- 3）调用类的静态方法

- 4）反射（Class.forName("com.lyj.load")）

- 5）初始化一个类的子类（会首先初始化子类的父类）

- 6）JVM启动时标明的启动类，即文件名和类名相同的那个类

只有这6中情况才会导致类的类的初始化。

类的初始化步骤：

   1）如果这个类还没有被加载和链接，那先进行加载和链接

   2）假如这个类存在直接父类，并且这个类还没有被初始化（注意：在一个类加载器中，类只能初始化一次），那就初始化直接的父类（不适用于接口）

   3）加入类中存在初始化语句（如static变量和static块），那就依次执行这些初始化语句。

22、类的加载

类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个这个类的Java.lang.Class对象，用来封装类在方法区类的对象

类的加载的最终产品是位于堆区中的Class对象
​        Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口

加载类的方式有以下几种：

- 1）从本地系统直接加载

- 2）通过网络下载.class文件

- 3）从zip，jar等归档文件中加载.class文件

- 4）从专有数据库中提取.class文件

- 5）将Java源文件动态编译为.class文件（服务器）

23、有序的map  LinkedHashMap

HashMap是无序的，HashMap在put的时候是根据key的hashcode进行hash然后放入对应的地方，但是当你要顺序获取值的时候，并不是按照put的顺序来得到的。

单纯的HashMap是无法实现排序的，Java在JDK 1.4之后，提供了LinkedHashMap来实现有序的HashMap。LinkedHashMap取键值对的时候是按照你放入的顺序来取的。

其基本用法与HashMap一样。Map maps = new LinkedHashMap();   maps.put("1", "张三");    

24、HashMap,LinkedHashMap,TreeMap的区别对比

- 1. HashMap里面存入的键值对在取出的时候是随机的,也是我们最常用的一个Map.它根据键的HashCode值存储数据,根据键可以直接获取它的值，具有很快的访问速度。在Map 中插入、删除和定位元素，HashMap 是最好的选择。 
- 2. TreeMap取出来的是排序后的键值对。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。 
- 3. LinkedHashMap 是HashMap的一个子类，如果需要输出的顺序和输入的相同,那么用LinkedHashMap可以实现.

4.加载器

来自http://blog.csdn.net/cutesource/article/details/5904501

JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：

- 1）Bootstrap ClassLoader

负责加载$JAVA_HOME中jre/lib/rt.jar里所有的class，由C++实现，不是ClassLoader子类

- 2）Extension ClassLoader

负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/*.jar或-Djava.ext.dirs指定目录下的jar包

- 3）App ClassLoader

负责记载classpath中指定的jar包及目录中class

- 4）Custom ClassLoader

属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader

加载过程中会先检查类是否被已加载，检查顺序是自底向上，从Custom ClassLoader到BootStrap ClassLoader逐层检查，只要某个classloader已加载就视为已加载此类，保证此类只所有ClassLoader加载一次。而加载的顺序是自顶向下，也就是由上层来逐层尝试加载此类。



#### javaScript知识点梳理

1、JS中的六种数据类型：boolean，string，number，null，undefined。

2、 
```js
 var one; 
 var two=null; 
 console.log(one==two,one ===two);    //true，false
```

one==two是loose comparison，比较的是值；one===two是strict comparison，既比较值，又比较类型。即

== equality 等同，=== identity 恒等
one的值是undefined，two的值为null。js中undefined的值派生自null值，所以规定undefined==null

3、盒子模型。详情按F12。

4、angularjs中的自定义服务有factory，service，provider。

5、JS中定义的函数会优先解析，而不是顺序解析（同步模式），并且若方法同名，后者会覆盖前者。

6、代码从上往下的一次执行，这种模式称之为同步。一部分先执行；另一部分在未来执行的模式称之为异步，如：ajax的post请求的callback回调函数。

7、Array数组对象：

   将已知的arr数组复制给新数组 ： `var a = arr.slice(0)`;

8、
```js
var obj={“key”：“1”，“value”：“2”}；
var newObj = obj;   //相当于两个对象指向同一个地址，
//修改其中任何一个，另一个也会受到影响
```

9、`$(emit)`是向上冒泡   ； `$broadcast()`是向下传播事件

10、NOSCRIPT标签用来定义在脚本未被执行时的替代内容

11、
```js
//已定义好一个checkState函数
windows.setTimeout(checkState(), 10000);  //立即被调用
windows.setTimeout(checkState, 10000);  //10s后被调用
windows.setTimeout(“checkState()”, 10000);  //10s后被调用，带引号的
```

12、

```java
console.log( 1+ "2" + "2");   //字符串相加等于字符串的合并，结果122
console.log( 1+ + "2" + "2");   //第一个+“2”中的加号是一元加操作符，会自动转成数值2，结果32                   
console.log( "A"- "B" + 2);//"A"- "B"先用Number将函数转换为数值，结果为NaN，在算数运算中有一个为NaN，则结果仍为NaN，结果NaN
console.log( "A"- "B" + "2");    //接上，合并后结果为NaN2
```

13、`Number()`函数会将对象的值转换为数字，null； empty ；0；默认都会转换成0

14、javaScript是弱类型语言，会根据后面的语言进行转换
```js
var a = “40”， b =7；
console.log(a%b);   //5
```

15、

```json
var obj = new Boolean('false');  //返回的是对象
var flag = Boolean(0);   //返回布尔值False
```

16、

```js
$post(url)是ajax请求；
ajax的事件是：ajaxComplete(callback);
             ajaxError(callback);
             ajaxSend(callback);
             ajaxStart(callback);
             ajaxStop(callback);
             ajaxSuccess(callback)            
```

17、
```js
function argsAsArray(fn, arr) {
    return fn.apply(this,arr);
}  //通过argsAsArray方法调用自生的fn方法，传入参数是arr
```
apply方法：   apply([thisObj[,argArray]])   ---- 应用某一对象的一个方法，用另一个对象替换当前对象。 参数是数组

　说明： 如果argArray 不是一个有效的数组或者不是 arguments 对象，那么将导致一个 TypeError。 

　　 如果没有提供 argArray 和 thisObj 任何一个参数，那么 Global 对象将被用作 thisObj， 并且无法被传递任何参数。

　　

call方法:    call([thisObj[,arg1[, arg2[,   [,.argN]]]]])   ----　调用一个对象的一个方法，以另一个对象替换当前对象。 参数是参数列表

   例子：A.call(B[,args1[,args2...]])    用A对象去替换B对象

　 说明： call 方法可以用来代替另一个对象调用一个方法。call 方法可将一个函数的对象上下文从初始的上下文改变为由 thisObj 指定的新对象。 
　　　　如果没有提供 thisObj 参数，那么 Global 对象被用作 thisObj。 

  借鉴他人的博客：http://uule.iteye.com/blog/1158829
　　　　
18、关于prototype原型　　

```js
//使用构造函数法
function createModule(str1, str2) {    
     function Obj(){       
     this.greeting = str1;       
     this.name = str2;        
     this.sayIt = function(){   
          return this.greeting + ', ' + this.name;  
       };  
  }   
    return new Obj();
}
//构造函数与原型组合
function createModule(str1, str2) {    
     function CreateMod(){   
         this.greeting = str1; 
        this.name = str2;  
   }    
     CreateMod.prototype.sayIt = function(){      
         return this.greeting + ', '  + this.name;   
   }   
         return new CreateMod();
         }            
```

19、toString()方法将对象转换成字符串。如果带参数toString(n)，即将其转换成n进制表示。将字符串转换为对象

```java
//1、使用JSONObject
JSONObject jsonObject=JSONObject.fromObject(objectStr);
Student stu=(Student)JSONObject.toBean(jsonObject, Student.class);
```

20、javascript标准事件模型的执行顺序：事件捕获->事件处理->事件冒泡。先事件捕获从windows到document往下级知道特定的事件节点，然后进行事件处理，在事件冒泡，

　　从特定节点往上级，完成整个过程。

21、页面性能指标：

- 白屏时间---用户从打开页面开始到也买你开始有东西呈现为止。

- 首屏时间---用户浏览器首屏内所有内容都呈现出来所花费的时间。

- 用户可操作时间（dom Interactive）---用户进行正常的点击、输入等操作，默认可以统计dom ready时间，因为通常会在这个时候绑定用户操作。

- 总下载时间---页面所有资源加载完成并呈现出来所花费的时间，即页面onload时间。

参照：http://www.cnblogs.com/chuaWeb/p/PerformanceMonitoring.html

22、javascript中，变量分为基本数据类型和引用数据类型两种，基本数据类型用8字节内存，存储基本数据类型（数值、布尔值、null和未定义）的值，引用类型的变量则只保存对对象、数组和函数等引用类型的值得引用（即内存地址）。javascript内部，所有数字都是以64位浮点数形式存储的。

 基本数据类型：Number、String、Boolean、Undefined、NUll

 复杂数据类型：Object、Array、Function、RegExp、Date、Error

 全局数据类型：Math

以上，除了NULL，其余的都可以成为JS的内置对象。

23、angular.js中的服务实质上是单例对象。单例模式有第三个特点：一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须向整个系统提供这个实例。

24、NaN，即非数值（Not a Number）是一个特殊的数值，这个数值用来表示一个本来要返回数值的操作数未返回数值的情况（这样就不会跑出错误了）。

25、一个promise可能有三种状态：等待（pending）、已完成（fulfiled）、已拒绝（rejected）。一个promise的状态只能从“等待”转向“完成”或者“拒绝”态，不能逆向转化，同时“完成”态和“拒绝”态不能相互转化promise实现then方法。（可以说，then是promise的核心），而且then必须返回一个promise，同一个promise的可以调用多次，而且回调的执行顺序和他们被定义时的顺序一致then方法接受两个参数，第一个参数是成功是的回调，在promise由“等待”状态转换到“完成”态是调用，另一个是失败时的回调，在promise由“等待”状态转到“拒绝”态时调用。

26、hasOwnProperty：用来判断一个对象是否有你给出的名称的属性或对象。但此方法无法检查该对象原型链中是否具有属性值，该属性必须是对象本身的一个成员。

   isPrototypeOf用来判断要检查其原型链的对象是否存在于指定对象实例中，是则返回true，否则返回false。

27、

```js
var f = function g(){
   return 23;
};
typeof g();     //结果是ReferenceError，g is not defined
typeof g;      //结果是undefined
typeof f();    //结果是number
typeof f;    //结果是function
```
注： typeof  null  的结果是  object

28、Jquery   siblings()方法返回被选元素的所有同胞元素；

29、回调时，被回调的函数会放在event loop里，等待线程里的任务执行完后才执行event loop里面的代码。

30、对元素的margin设置百分数，是相对于父元素width计算的。

31、Html中的标签分为闭合标签和自闭合标签。自闭和标签有<input/><img/><br/><link/><hr/>等。

32、<hr/>定义水平线。

33、Css样式中的权值大小：第一等：内联样式，如：“style= ”，权值为1000

   第二等：ID选择器，如：#content，权值为0100

   第三等：类，伪类和属性选择器，如：.content，权值为0010

   第四等：代表类型选择器和伪元素选择器，如：div  p，权值为0001

   通配符、子选择器、相邻选择器等，如：*、>、+，权值为0000 ；  继承的样式没有权值

34、HTML body部分中的javascript会在页面加载的时候被执行；head部分中的只有在被调用的时候才会被执行。

35、不换行必须设置word-break:break-all 处理单词的打断 和 word-spacing:no-wrap处理元素内的空白，只在一行内显示。

36、link是在加载页面前把Css加载完毕；@import url()是在读取文件文件后加载，所以会出现一开始没有css样式，闪烁一下出现样式后的页面（网速慢的情况下）

37、CSS样式：边距：10px 20px 30px 40px哪一个是底边距？  巧计：顺时针，上右下左。

38、parseInt方法可以将其它进制转换为十进制。

39、基本类型和封装类型进行“==”运算符比较时，封装型会自动拆箱为基本型在比较。

   两个Integer类型进行“==”比较，如果其值在-128至127，那么返回true，否则返回false。

　两个封装型进行equals比较，会先比较类型在比较值。

```js
int a=257；
Integer b=257；
Integer c=257；
Integer b2=257；
Integer c2=257；
System.ou.println(a==b);
//System.ou.println(a.equals(b)); //编译出错，基本类型不能调用equals
System.ou.println(b.equals(257.0));
System.ou.println(b==c);
System.ou.println(b2==c2);

//上面的代码结果依次为true，false，false，true
```


40、`suspend()`使得线程进入阻塞状态，并且不会自动回复。

　　 `resume()`使进程重新进入可执行状态。（这两个配套使用）

41、6种基本数据类型：byte，bollean，short，char，int，long，float，double。

42、自动数据类型转换：按照由低到高的顺序转换。优先关系如下：

   byte，short，char--> int--> long --> float --> double

  其中，最底层的类型不能进行运算，若有运算会自动向上转换int型。

43、根类Object中的方法：clone();equals();finalize();getClass();notify();notifyAll();hashCode();toString();wait()

44、
```js
var a=2；
var b=3;
if(a=b){...
}     //在java中“=”赋值是有返回值的，赋什么值就返回什么值
      //C中赋值后会与0进行比较，大于0返回true，否则false
```

45、String类是不可改变的类，String str =“123”；str=“1”并不是覆盖而是新建一个内存空间指向它

46、解决哈希冲突的两种方法：开放地址法、链地址法

47、常见的浏览器端存储技术： cookie  、 sessionstorage   、localstorage

48、在HTML body部分中的JavaScripts会在页面加载的时候被执行。 在HTML head部分中的JavaScripts会在被调用的时候才执行

49、javascript中在进行算术运算时，“+号”，数字隐式转换成字符串。其余的运算符号是字符串隐式转换成数字。

#### 其他

1. 在计算机内部数据的存储和运算都采用二进制；
2. 计算机中数据分为有符号数和无符号数，对于有符号数，计算机规定用最高位来表示符 号。“0”表示正数，“1”表示负数；
3. Java中的数据都是有符号数；
4. 计算机中带符号的整数都是使用二进制的补码。




#### Bug案例集中营

##### 1、get方法提交表单，后台出现中文乱码问题  (注明:web.xml中的编码设置只是针对post请求)

   在eclipse中，找到tomcat的目录，在这个目录下面\tomcat61-32\tomcat61\conf，找到server.xml

   打开文件，找到这一行  <Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>

   将其修改成这样 <Connector URIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>

   最后清理服务器缓存

   （如不行，更改tomcat原始路径的配置）

   注：配置useBodyEncodingForURI="true"后，可以解决普通get请求的中文乱码问题

　　 但是对于通过ajax发起的get请求中文依然会乱码，请把useBodyEncodingForURI="true"改为URIEncoding="UTF-8"即可

##### 2、Js的数组排序sort()，默认使用字母数字（字符串的Unicode码）排序。例如：[1，2，5，10].sort()会输出 [1,10,2,5] 。正确的排序可用 [1，2，5，10].sort(a,b)

##### 3、JSp使用include标签包含子页面时出现乱码。方法：在子页面的顶部加入：<%@page pageEncoding="UTF-8"%> 

##### 4、Linux的权限读取文件，java往服务器上上传文件后，再次登录就没有权限了。

##### 5、写接口调试的时候，浏览器查看乱码，解决办法：

- 1.通过代码查看当前电脑的编码类型System.out.println(System.getProperty("file.encoding")); 或者 System.out.println(Charset.defaultCharset());，任意选一种即可。

- 2.response.setCharacterEncoding("utf-8");//第一句，设置服务器端编码
   response.setContentType("text/html;charset=utf-8");//第二句，设置浏览器端解码

##### 6、linux系统下mysql表名区分大小的问题：  

 1、数据库名与表名是严格区分大小写的；2、表的别名是严格区分大小写的；3、列名与列的别名在所有的情况下均是忽略大小写的；4、变量名也是严格区分大小写的；
 注：MySQL在Windows下都不区分大小写。
　　可以查看Linux上的MySQL的配置文件/etc/my.cnf ，就需要在[mysqld]下面添加一行配置，即 lower_case_table_names=1: （其中 0：区分大小写，1：不区分大小写）

```powershell
[root@VM_219_131_centos tomcat7]# vi /etc/my.cnf 
[mysqld]
lower_case_table_names=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

补：如果想在查询时区分字段值的大小写，则：字段值需要设置BINARY属性，设置的方法有多种：

- A、创建时设置：
```sql
CREATE TABLE T(

A VARCHAR(10) BINARY

);
```
- B、使用alter修改：

`ALTER TABLE`tablename` MODIFY COLUMN `cloname` VARCHAR(45) BINARY;`

- C、mysql table编辑时中直接勾选BINARY项。
```powershell
## 修改完后，一定记得重启数据库。
[root@VM_219_131_centos tomcat7]# service mysqld restart
Stopping mysqld:  [  OK  ]
Starting mysqld:  [  OK  ]
```

### AOP面向切面编程
- @Apsect：将当前类标识为一个切面；
- @Pointcut：定义切点，这里使用的是条件表达式；
- @Before：前置增强，就是在目标方法执行之前执行；
- @AfterReturning：后置增强，方法退出时执行；
- @AfterThrowing：有异常时该方法执行；
- @After：最终增强，无论什么情况都会执行；
- @Afround：环绕增强；
- 
所谓AOP也就是面向切面编程，能够让我们在不影响原有业务功能的前提下，横切扩展新的功能。这里面有一个比较显眼的词我们需要注意一下，横切，它是基于横切面对程序进行扩展的。
```java
@Aspect
@Component
public class LogAspect {
   /**
    * 功能描述: 拦截对这个包下所有方法的访问
    *
    * @param:[]
    * @return:void
    **/
   @Pointcut("execution(* com.example.springbootaop.controller.*.*(..))") //指明要横切扩展的路径
   public void loginLog() {
   }

   // 前置通知
   @Before("loginLog()")
   public void loginBefore(JoinPoint joinPoint) {
       // 我们从请求的上下文中获取request，记录请求的内容
       ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
       HttpServletRequest request = attributes.getRequest();
       System.out.println("请求路径 : " + request.getRequestURL());
       System.out.println("请求方式 : " + request.getMethod());
       System.out.println("方法名 : " + joinPoint.getSignature().getName());
       System.out.println("类路径 : " + joinPoint.getSignature().getDeclaringTypeName());
       System.out.println("参数 : " + Arrays.toString(joinPoint.getArgs()));
   }

   @AfterReturning(returning = "object", pointcut = "loginLog()")
   public void doAfterReturning(Object object) {
       System.out.println("方法的返回值 : " + object);
   }

   // 方法发生异常时执行该方法
   @AfterThrowing(throwing = "e",pointcut = "loginLog()")
   public void throwsExecute(JoinPoint joinPoint, Exception e) {
       System.err.println("方法执行异常 : " + e.getMessage());
   }

   // 后置通知
   @After("loginLog()")
   public void afterInform() {
       System.out.println("后置通知结束");
   }

   // 环绕通知
   @Around("loginLog()")
   public Object surroundInform(ProceedingJoinPoint proceedingJoinPoint) {
       System.out.println("环绕通知开始...");
       try {
           Object o =  proceedingJoinPoint.proceed();
           System.out.println("方法环绕proceed，结果是 :" + o);
           return o;
       } catch (Throwable e) {
           e.printStackTrace();
           return null;
       }
   }
}
```