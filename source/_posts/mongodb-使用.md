---
layout: post
title: mongodb-使用
date: 2018-08-06 06:45:50
tags: mongodb
categories: mongodb
---

### 安装与启动
官网下载.msi,按照提示完成安装。
在安装目录的 `data`文件夹下，新建db文件夹，用于存放数据。默认为：`C:\data\db\`

```shell
mongod.exe --dbpath "D:\Program Files\MongoDB\Server\4.0\data\db"
```

正常启动会出现：
![](http://p2jr3pegk.bkt.clouddn.com/mongo01-1.png)

> 可以定义.bat文件，然后输入上面内容，避免每次指明db路径，建议在添加完用户后，指明--auth开启权限认证功能

<!-- more -->

也可通过浏览器访问http://localhost:27017/，页面上输出：

```
It looks like you are trying to access MongoDB over HTTP on the native driver port.
```

保持服务后台运行，在`bin`目录下，打开mongo.exe打开终端窗口，完成一些操作。

当然，你可以将mongo的安装目录下的`bin`目录添加到环境变量中，这样就可以直接在命令行中启动服务了。但是需特别注意，一定要先启动mongod(服务端程序)，且保持服务一直运行，然后在启动mongo(客户端程序)完成一些操作。

### Bug
1、闪退问题(原因不存在数据存储路径)
>在某一个位置创建文件夹，此处我设置的为D盘mongoData文件夹。打开cmd，进入到MongoDB的bin目录下，输入mongod --dbpath D:/mongoData即可修复

### 基本操作
默认会连接到test数据库
```shell
> show dbs       #查看当前有哪些数据库
> db              #查看当前所在的数据库
> use admin       #切换到 admin 数据库
> show collections      #查看当前数据库中有哪些集合
> user admin; show users;   #查看admin库下的所有用户
> db.system.users.find();  #查看所有用户，必须是在admin库下才能看到
> db.users.insert({username:"george",pwd:"1234"})   #创建了users这个集合，并往其中插入一条数据;另一种实现方式为 save
> db.users.find()      #查看users集合的所有数据
> db.users.update({username:"george"},{age:22})   #修改数据，修改的是整个文档，会替换掉数据
> db.users.update({username:"george"},{$set:{pwd:"9999"}})   #修改数据，部分修改，只会修改对应的字段值
> db.users.find({username:"george"})      #条件查找
> db.users.drop()      #删除users集合
> db.users.remove({username:"george"})     #删除某一条数据
> db.users.getIndexes()     #查看索引
> db.users.ensuerIndex({username:1})    #创建索引，1表示正序索引，-1表示逆序索引
> db.users.dropIndex({username:1})     #删除索引
> db.users.find().pretty()       #格式化显示数据
> db.users.find({key : {$gt:value}})      #查看users这个集合中，key>value的数据。类似的还有：（1）大于等于: {key : {$gte:value} }、（2）小于：{key : {$lt:value} }、（3）小于等于：{key : {$lte:value}}、（4）不等于：{key : {$ne:value} }、（5）且操作AND： {key1:value1, key2:value2, key3:value3 …} (6)或操作OR： {$or: [{key1: value1}, {key2:value2} ]} 
> db.users.find({likes:{$gte:888},$or:[{username:"mongo"},{username:"spring"}]})          #复杂查询语句，类似sql语句 like>=888 && (username=”mongo” or username=”spring”)
```
>需要特别注意的是，insert和save方法中如果包含"_id"字段，save方法会覆盖掉之前的数据，而insert则不会插入成功。
>find()语法：find({},{KEY:1/0}) find的第二个参数，KEY为要显示或隐藏的字段，value为1表示显示，0表示隐藏，默认为1。find()方法后面还可以跟 limit(n), skip(n), sort({key:value}),分别实现限制返回的结果条数、跳过多少条数据和排序功能。sort({key:value,...})语法中，key表示要排序的字段，value的可取值为 1 / -1 。1表示升序asc，-1表示降序desc

### 开启安全认证
启动服务时，开启安全认证：
```shell
mongod --auth
```
启动客户端，发现仍可以连接，但是无法操作。此时需要在admin数据库中创建用户,语法如下：
```json
db.createUser({
    user:"username",
    pwd:"password",
    customData:{any info},
    roles:[{role:"<role>",db:"<db>"},{role:"<role>",db:"<db>"}]
})
```
添加成功后，会显示Json格式的信息以及success的成功信息
>mongodb内建的角色: read, readWrite, dbAdmin, dbOwner, userAdmin，dbAdminAnyDatabase，userAdminAnyDatabase，readWriteAnyDatabase，readAnyDatabase，clusterAdmin

#### 操作
这里创建用户：root，密码：root，权限：只读权限
```shell
> db.createUser({user:"root",pwd:"root",roles:[{role:"read",db:"demo"}]})
```
然后进行认证,以后都需要进行权限认证
```shell
>db.auth("root","root")
```
>注意，查看添加的用户信息在admin表中查看，创建了demo库，但是没有显示，是因为没有数据，向其中茶树一条数据即可完成显示。还需要注意的是插入操作要先选择库。

#### 远程连接
为了以后方便，在配置文件mongod.cfg中设置：
```yaml
security:
  authorization: enabled
```
重新启动服务端，然后通过远程连接
```shell
>mongo admin -u root -p root    //连接到Mongo数据库
```


### 开启监控
1、自带的功能：

```powershell
$ mongostat
```
或者：
```powershell
$ mongotop
```
或者(启动的时候添加--rest参数，开启28017端口，然后通过浏览器访问):
```shell
./mongod --dbpath ../data/db/ --rest
```

2、自带的数据库状态查看：

```shell
>db.serverStatus()
>db.stats()
>db.collection.stats()
```

参考：[博客](https://blog.csdn.net/u013614451/article/details/48900867)

### 非关系型数据库:Mongdb

1.创建数据库：use DATABASE_NAME；

2.查看数据库： show DATABASE_NAME;

3.删除数据库：db.dropDatabase()；

4.删除集合：db.collection.drop()；

5.插入文档：db.COLLECTION_NAME.insert(document)；

6.更新文档：db.collection.update(

<query>,
<update>,

{ upsert: <boolean>,

multi: <boolean>,

writeConcern: <document>

})；

参数说明：
- query : update的查询条件，类似sql update查询内where后面的。
- update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
writeConcern :可选，抛出异常的级别。

7.传入的文档来替换已有文档（保存） ：db.collection.save(
<document>,
{
　　　writeConcern: <document>
} )；

参数说明：

`document : 文档数据`。
`writeConcern :可选，抛出异常的级别`。


更多实例:

只更新第一条记录：
```
db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );
```
全部更新：
```
db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );
```
只添加第一条：
```
db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );
```
全部添加加进去:
```
db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );
```
全部更新：
```
db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );
```
只更新第一条记录：
```
db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );
```

8.删除文档：db.collection.remove(
<query>,
{
justOne: <boolean>,
writeConcern: <document>
}
)
参数说明：

- query :（可选）删除的文档的条件。
- justOne : （可选）如果设为 true 或 1，则只删除一个文档。
- writeConcern :（可选）抛出异常的级别。

9.查询文档：db.collection.find(query, projection)；

query ：可选，使用查询操作符指定查询条件
projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）

注：若要用以易读（格式化）的方式来显示文档：db.col.find().pretty()；

          多条件查询（AND）：db.col.find({key1:value1, key2:value2}).pretty()
    
          或查询（Or）：db.col.find(

　　　　　　　　　　　　{

　　　　　　　　　　　　　　$or: [

　　　　　　　　　　　　　　　　{key1: value1}, {key2:value2}

　　　　　　　　　　　　　　　　]

　　　　　　　　　　　　}

　　　　　　　　　　　).pretty()；          

10.清空结合：db.col.remove({})；

11.MongoDB中条件操作符有：

(>) 大于 - $gt
(<) 小于 - $lt
(>=) 大于等于 - $gte
(<= ) 小于等于 - $lte