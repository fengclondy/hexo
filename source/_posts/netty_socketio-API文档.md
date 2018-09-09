---
layout: post
title: netty-socketio API文档
date: 2018-04-09 03:15:22
tags: netty-socketio
categories: netty-socketio
---

>netty-socketio是基于netty的socket.io服务实现，可以无缝对接前端使用的socketio-client.js。 
相对于javaee的原生websocket支持（@serverEndpoint）和spring-boot的MessageBroker(@messageMapping)，netty-socketio绝对是最好用的websocket后台实现。
因为netty-socketio完整的实现了socket.io提供的监听前台事件、向指定客户端发送事件、将指定客户端加入指定房间、向指定房间广播事件、客户端从指定房间退出等操作。


### 基本变量和函数实现

#### SocketIOClient 

- joinRoom() ---加入到指定房间。
- leaveRoom() ---从指定房间离开。
- getSessionId() ---返回由UUID生成的唯一标识。
- getAllRooms() ---返回当前客户端所在的room名称列表。
- sendEvent(eventname,data) ---向当前客户端发送事件。


#### SocketIOServer 

- getAllClients() ---返回默认名称空间中的所有客户端实例。
- getBroadcastOperations() ---返回默认名称空间的所有实例组成的广播对象。
- getRoomOperations() ---返回所有命名空间中指定房间的广播对象，如果命名空间只有一个，该方法到可以大胆使用。
- getClient(uid) ---返回默认名称空间的指定客户端。
- getNamespace() ---返回指定名称的命名空间。

<!-- more -->


#### Namespace 命名空间

- getAllClients()            ---获得本namespace中的所有客户端。
- getClient()               ---获得指定id客户端对象。
- getRoomClients(room)             ---获得本空间中指定房间中的客户端。
- getRooms()              ---获得本空间中的所有房间。
- getRooms(client)      ---获得指定客户端所在的房间列表。
- leave(room,uuid) ---将指定客户端离开指定房间，如果房间中已无客户端，删除该房间。
- getBroadcastOperations ---返回针对空间中所有客户端的广播对象。
- getRoomOperations(room) ---返回针对指定房间的广播对象。


#### BroadcastOperations 广播操作对象

- sendEvent(eventname,data) ---向本广播对象中的全体客户端发送广播。
- sendEvent(eventname,excludeSocketIOClient,data) ---排除指定客户端广播


### 数据结构及注意事项

1、数据结构

```java
 public void getUserStatus(SocketIOClient client, AckRequest request, MsgInfo data) {
 }
```

>SocketIOClient是当前触发该方法执行的客户端；AckRequest是客户端是否需要回调的函数（比如返回一个状态码200）；
MsgInfo是定义好的实体类，可直接使用set和get方法。

2、注意事项：

- 接受的参数个数不限定为一个，可以多个

```java
 public void getUserStatus(SocketIOClient client, AckRequest request, MsgInfo data,String roomId ...) {
 }
```

- 回调函数有无参和有参之分

```java
 client.sendEvent("chatMsgReceive", new VoidAckCallback(100) {  //100s超时触发，可以不加
     protected void onSuccess() {
     System.out.println("1. : " + client.getSessionId());
     },
     public void onTimeout() {
     }
 }, data);

//上面是接受无参数的客户端回调，下面是有参数的回调
 client.sendEvent("chatMsg", new AckCallback<String>(String.class) {
     @Override
     public void onSuccess(String result) {
          System.out.println("ack from client: " + client.getSessionId() + " data: " + result);
     }
 }, data);
```

- 如果传入的参数不是定义好的实体类型，netty-socketio默认的接受参数是序列化后的值，如果需要从里面取具体的对象，需要进行反序列化操作：

```java
//方式1，传入为list集合
public void chatMsgSend(SocketIOClient client, AckRequest request, List<MsgInfo> listData) {

     //将linkedhashMap转成object（反序列化）
     ObjectMapper mapper = new ObjectMapper();
     List<MsgInfo> infos = mapper.convertValue(listData, new TypeReference<List<MsgInfo>>(){});
}

//方式2，传入为一个对象
public void getOfflineMsg(SocketIOClient client, AckRequest request, Date dateTime) {
//将对象p转成object（反序列化）
     ObjectMapper mapper = new ObjectMapper();
     Date date = mapper.convertValue(dateTime, Date.class);
}
```

### 关于心跳包

客户端会定期发送心跳包，并触发一个ping事件和一个pong事件,服务器端需设置两个重要额参数：

```java
 // Ping消息超时时间（毫秒），默认60秒，这个时间间隔内没有接收到心跳消息就会发送超时事件(断开连接事件)
 config.setPingTimeout(60000);
 // Ping消息间隔（毫秒），默认25秒。客户端向服务器发送一条aishi心跳消息间隔
 config.setPingInterval(25000);
```

也就是说握手协议的时候，客户端从服务器拿到这两个参数，一个是ping消息的发送间隔时间，一个是从服务器返回pong消息的超时时间， 客户端会在超时后断开连接。
心跳包发送方向是客户端向服务器端发送，以维持在线状态。

#### 关于断线和超时

关闭浏览器、直接关闭客户端程序、kill进程、主动执行disconnect方法都会导致立刻产生断线事件。 
而客户端把网络断开，服务器端在 pingTimeout ms后产生断线事件、客户端在 pingTimeout ms后也产生断线事件。

实际上，超时后会产生一个断线事件，叫”disconnect”。客户端和服务器端都可以对这个事件作出应答，释放连接。

### 其他注意点

1、同一个浏览器，即使打开了两个不同的用户页面，也默认是同一个用户，因为用户的区分只与sessionId有关。
向目标主机发消息时，需要知道他的客户端sessionId。


参考：https://www.cnblogs.com/wxgblogs/p/6852470.html

https://www.xncoding.com/2017/07/16/spring/sb-socketio.html

