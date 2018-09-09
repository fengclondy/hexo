---
layout: post
title: socketio编程实战
date: 2018-04-27 02:29:49
tags: socketio
categoriets: socketio
---


#### 一、基础

- 1、HTTP协议是无状态的，服务器只会响应来自客户端的请求，但是它与客户端之间不具备持续连接；且只能从客户端主动请求服务端，服务端不能主动通知客户端。

 对于实时通信系统（聊天室或监控系统）这样显然是不合理的。传统的方法有：长轮询（客户端每隔很短的时间，都对服务器发出请求，当时间足够小就能实现实时的效果）、长连接（客户端只请求一次，但是服务器会将连接保持，当有数据时就返回结果给客户端）。这两种方式，都对客户端和服务器都造成了大量的性能浪费，于是WebSocket应运而生。WebSocket协议能够让浏览器和服务器全双工实时通信，互相的，服务器也能主动通知客户端。

- 2、 WebSocket的原理非常的简单：利用HTTP请求产生握手，HTTP头部中含有WebSocket协议的请求，所以握手之后，二者转用TCP协议进行交流。

 Socket.IO是业界良心，新手福音。它屏蔽了所有底层细节，让顶层调用非常简单。并且还为不支持WebSocket协议的浏览器，提供了长轮询的透明模拟机制。

 
 

#### 二、实现

- 1、socket.emit(action,arg1,arg2); 表示发送了一个action命令，还有两个数据，在另一端接收时，可以这么写： socket.on('action',function(arg1,arg2){...});

- 2、io.sockets.emit    信息传输对象为所有 client ; socket.emit 信息传输对象为当前 socket 对应的 client ，各个client socket 相互不影响;

socket.broadcast.emit 信息传输对象为所有 client ，排除当前socket 对应的 client 

- 3、在使用Node的http模块创建服务器同时还要Express应用，因为这个服务器对象需要同时充当Express服务和Socket.io服务。(如下)

```
var app = require('express')(); //Express服务
var server = require('http').Server(app); //原生Http服务
var io = require('socket.io')(server); //Socket.io服务
io.on('connection', function(socket){
    /* 具体操作 */
});
server.listen(3000);
当客户端需要连接服务器时，它需要先建立一个握手。io.处理连接事件，socket 处理断开连接事件。在上面代码里，这套握手机制是完全自动的，我们可以通过也可以io.use()方法来设置这一过程。
客户端使用js调用socket.io的Client API即可。
复制代码
<script src="/lib/socket.io/socket.io.js"></script>
<script>
   var socket = io();
   socket.on('connect', function() {
           /* 具体操作 */
   });
</script>
复制代码
4、同一个服务器可以使用namespaces创造不同的Socket连接。Socket.IO使用of()来指定不同的命名空间。

io.of('/someNamespace').on('connection', function(socket){
    socket.on('customEvent', function(customEventData) {
        /* 具体操作 */
    });
});
io.of('/someOtherNamespace').on('connection', function(socket){
    socket.on('customEvent', function(customEventData) {
    /* 具体操作 */
    });
});
　　服务器端则通过在定义Socket对象时传递namespace参数。
<script>
 var someSocket = io('/someNamespace');
 someSocket.on('customEvent', function(customEventData) {
     /* 具体操作 */
 });
 var someOtherSocket = io('/someOtherNamespace');
 someOtherSocket.on('customEvent', function(customEventData) {
     /* 具体操作 */
 });
</script>
　　在每一个namespace中又可以使用room来进一步划分，不过sockets是使用join()、leave()来调用。

//服务器端
io.on('event', function(eventData){
     //监听join事件
     socket.on('join', function(roomData){
          socket.join(roomData.roomName);
     });
     //监听leave事件
     socket.on('leave', function(roomData){
          socket.leave(roomData.roomName);
     });
});
//浏览器端
io.on('connection', function(socket){
     //在此room下触发事件
     io. in('someRoom') .emit('customEvent', customEventData);
});
```