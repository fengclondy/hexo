---
layout: post
title: springBoot整合netty-socketio的完整例子
date: 2018-04-04 09:20:29
tags: netty-socketio
categories: netty-socketio
---

### 技术选型

采用springBoot 1.5版本搭建，用到了token校验，redis缓存。

### 主要的代码结构

![](http://p2jr3pegk.bkt.clouddn.com/netty01.png)

<!-- more -->

#### 关键代码 

redisConfig

```java
package com.njnsi.netty.config;

import com.corundumstudio.socketio.*;
import com.corundumstudio.socketio.annotation.SpringAnnotationScanner;
import com.corundumstudio.socketio.listener.ExceptionListener;
import com.njnsi.netty.handler.MessageEventHandler;
import com.njnsi.netty.utils.JWTUtil;
import io.netty.channel.ChannelHandlerContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;

import java.util.List;

/**
 * NettyConfig class
 *
 * @author gqsu
 * @date 2018/04/05
 */
@org.springframework.context.annotation.Configuration
public class NettyConfig {

    private Logger log = LoggerFactory.getLogger(MessageEventHandler.class);

    @Value("${socketio.server.host}")
    private String host;

    @Value("${socketio.server.port}")
    private Integer port;

    @Bean
    public SocketIOServer socketIOServer()
    {
        /**
         * 创建Socket，并设置监听端口
         */
        Configuration config = new Configuration();
        /** 设置主机名*/
        config.setHostname(host);
//        /** 注意如果开放跨域设置，需要设置为null而不是"*" */
//        config.setOrigin(null);
        /** 设置监听端口*/
        config.setPort(port);
        /** 协议升级超时时间（毫秒），默认10秒。HTTP握手升级为ws协议超时时间*/
        config.setUpgradeTimeout(10000);
        /** Ping消息超时时间（毫秒），默认60秒，这个时间间隔内没有接收到心跳消息就会发送超时事件(断开连接事件)*/
        config.setPingTimeout(60000);
        /** Ping消息间隔（毫秒），默认25秒。客户端向服务器发送一条aishi心跳消息间隔 */
        config.setPingInterval(25000);

        /** 异常监听事件，必须覆写全部方法 */
        config.setExceptionListener(new ExceptionListener(){
            @Override
            public void onConnectException(Exception e, SocketIOClient client) {
                log.error(client.getRemoteAddress()+"，连接异常:"+e);
                client.sendEvent("exception","连接异常！");
            }
            @Override
            public void onDisconnectException(Exception e, SocketIOClient client) {
                log.error(client.getRemoteAddress()+"，断开异常:"+e);
                client.sendEvent("exception","断开异常！");
            }
            @Override
            public void onEventException(Exception e, List<Object> data, SocketIOClient client) {
                log.error(client.getRemoteAddress()+"，服务器异常:"+e+"，传入数据:"+data);
                client.sendEvent("exception","服务器异常");
            }
            @Override
            public void onPingException(Exception e, SocketIOClient client) {
                log.error(client.getRemoteAddress()+"，ping超时异常:"+e);
                client.sendEvent("exception","ping超时异常！");
            }
            @Override
            public boolean exceptionCaught(ChannelHandlerContext ctx, Throwable e) {
                return false;
            }
        });


        /** 连接认证，这里使用token更合适 */
        config.setAuthorizationListener(new AuthorizationListener() {
            @Override
            public boolean isAuthorized(HandshakeData data) {
                 /** http://localhost:8081?username=test&password=test */
                 /**例如果使用上面的链接进行connect，可以使用如下代码获取用户密码信息返回token */
                 String token = data.getSingleUrlParam("token");
                 String userName = JWTUtil.getUsername(token);
                 /** return JWTUtil.verify(token,userName,"111111"); */
                return true;
            }
        });
        return new SocketIOServer(config);
    }

    @Bean
    public SpringAnnotationScanner springAnnotationScanner(SocketIOServer socketServer) {
        return new SpringAnnotationScanner(socketServer);
    }
}

```

MessageEventHandler(有删减)

```java
package com.njnsi.netty.handler;

import java.util.*;
import com.corundumstudio.socketio.*;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.njnsi.netty.domain.MsgInfo;
import com.njnsi.netty.domain.MsgRoomInfo;
import com.njnsi.netty.repository.MsgInfoRepository;
import com.njnsi.netty.repository.MsgRoomInfoRepository;
import com.njnsi.netty.utils.JWTUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.redis.core.*;
import org.springframework.stereotype.Component;
import com.corundumstudio.socketio.annotation.OnConnect;
import com.corundumstudio.socketio.annotation.OnDisconnect;
import com.corundumstudio.socketio.annotation.OnEvent;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * MessageEventHandler class
 *
 * @author gqsu
 * @date 2018/04/05
 */
@Component
public class MessageEventHandler{

    private Logger logger = LoggerFactory.getLogger(MessageEventHandler.class);

    private final SocketIOServer server;

    /**定义发送超时时间60s*/
    private int timeOut= 60;

    /**定义缓存入库的时间*/
    private long diffTime =10*60*1000;

    /**定义消息有效撤回的时间*/
    private long callbackTime =10*60*1000;

    /** 定义在线/忙碌/离线状态标识*/
    private String onlineStatus = "1";
    private String busyStatus = "2";
    private String offlineStatus = "3";

    /**定义返回给客户端的状态码*/
    private int resCode;

    @Autowired
    private MsgInfoRepository msgInfoRepository;

    @Autowired
    private MsgRoomInfoRepository msgRoomInfoRepository;

    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    public MessageEventHandler(SocketIOServer server) {
        this.server = server;
    }

    @PostConstruct
    public void init() {
        server.start();
    }

    @PreDestroy
    public void close() {
        server.stop();
    }

    /**添加connect事件，当客户端发起连接时调用*/
    @OnConnect
    public void onConnect(SocketIOClient client) {

        String token = client.getHandshakeData().getSingleUrlParam("token");
        String userName = JWTUtil.getUsername(token);

        boolean exist = redisTemplate.hasKey(userName);
        if(!exist){
            /**以（用户名-sessionid）形式存入缓存,用于定向消息发送*/
            redisTemplate.opsForValue().set(userName, client.getSessionId().toString());
            /**存入缓存，当前在线人员*/
            redisTemplate.opsForSet().add("onlineUsers",userName);
            client.sendEvent("connectRes", "200");
        }else{
            client.sendEvent("connectRes", "405");
        }

        /**全局广播当前在线人员和忙碌人员名单*/
        logger.info("在线用户人数：" +  redisTemplate.opsForSet().members("onlineUsers"));
        logger.info("忙碌用户人数：" +  redisTemplate.opsForSet().members("busyUsers"));

        server.getBroadcastOperations().sendEvent("onlineUsers",  redisTemplate.opsForSet().members("onlineUsers"));
        server.getBroadcastOperations().sendEvent("busyUsers",  redisTemplate.opsForSet().members("busyUsers"));
    }


    /**添加@OnDisconnect事件，客户端断开连接时调用，刷新客户端信息*/
    @OnDisconnect
    public void onDisconnect(SocketIOClient client) {

        String token = client.getHandshakeData().getSingleUrlParam("token");
        String userName = JWTUtil.getUsername(token);

        redisTemplate.opsForValue().set("time_"+userName+"",new Date());

        boolean exist = redisTemplate.hasKey(userName);
        if(exist){
            redisTemplate.delete(userName);
            redisTemplate.opsForSet().remove("onlineUsers",userName);
            redisTemplate.opsForSet().remove("busyUsers",userName);
            redisTemplate.delete("content_"+userName+"");
            logger.info(userName+"下线成功！");
            client.disconnect();
        }else{
            logger.error("登录超时，缓存信息已过期!已下线");
        }

        logger.info("在线用户人数：" + redisTemplate.opsForSet().members("onlineUsers"));
        server.getBroadcastOperations().sendEvent("onlineUsers",  redisTemplate.opsForSet().members("onlineUsers"));
        server.getBroadcastOperations().sendEvent("busyUsers",  redisTemplate.opsForSet().members("busyUsers"));
    }


    /**消息接收入口，当接收到消息后，查找发送目标客户端，并且向该客户端发送消息*/
    @OnEvent(value = "chatMsgSend")
    public void chatMsgSend(SocketIOClient client, AckRequest request, List<MsgInfo> listData){

        int hasSend = 0;
        /**将linkedhashMap转成object（反序列化）*/
        ObjectMapper mapper = new ObjectMapper();
        List<MsgInfo> infos = mapper.convertValue(listData, new TypeReference<List<MsgInfo>>(){});

        String fromUser = infos.get(0).getFromUser();
        String toUser = infos.get(0).getToUser();
        Date date = infos.get(0).getDate();

        String sessionId =  redisTemplate.opsForValue().get(fromUser).toString();
        boolean exist1 = client.getSessionId().toString().equals(sessionId);
        if (exist1) {
            boolean exist2 = redisTemplate.hasKey(toUser);
            /**在线状态(在线和忙碌)*/
            if (exist2) {
                UUID uid = UUID.fromString(redisTemplate.opsForValue().get(toUser).toString());
                server.getClient(uid).sendEvent("chatMsgReceive", new VoidAckCallback(timeOut) {
                    @Override
                    protected void onSuccess() {
                        logger.info("2.确认接收方收到服务器下发的在线消息: ");
                        /**缓存接收者的接收时间*/
                        redisTemplate.opsForValue().set("time_"+toUser+"",new Date());
                    }
                    @Override
                    public void onTimeout() {
                        server.getClient(uid).sendEvent("chatMsgReceive",infos);
                    }
                }, infos);

                boolean res = redisTemplate.hasKey("content_"+toUser+"");
                /**接收者忙碌*/
                if(res){
                    String content = redisTemplate.opsForValue().get("content_"+toUser+"").toString();
                    /**构造信息*/
                    List<MsgInfo> list = new ArrayList<>();
                    MsgInfo msg = new MsgInfo();
                    msg.setContent(content);
                    msg.setDate(date);
                    msg.setFromUser(toUser);
                    msg.setToUser(fromUser);
                    msg.setMsgType("text");
                    list.add(msg);
                    /**先回调客户端*/
                    if (request.isAckRequested()) {
                        request.sendAckData(resCode);
                    }
                    client.sendEvent("chatMsgReceive", new VoidAckCallback(timeOut) {
                        @Override
                        protected void onSuccess() {
                            logger.info("2.确认发送方收到服务器下发的忙碌消息: ");
                            /**缓存接收者的接收时间*/
                            redisTemplate.opsForValue().set("time_"+toUser+"",new Date());
                        }
                        @Override
                        public void onTimeout() {
                            client.sendEvent("chatMsgReceive",list);
                        }
                    }, list);
                    hasSend = 1;
                    redisTemplate.opsForSet().add("msg",msg);
                }
                resCode = 200;
            } else {
                resCode = 201;
                logger.info("接收方离线！ ");
            }
        } else {
            logger.info("非法操作，不是已登录的客户端 ");
            resCode = 401;
        }
        /**客户端的回调函数，用于判断是否消息发送成功（传回给发送方）*/
        if (request.isAckRequested() && hasSend ==0) {
            request.sendAckData(resCode);
        }
        /**缓存发送者的发送时间*/
        redisTemplate.opsForValue().set("time_"+fromUser+"",new Date());
        /**缓存消息*/
        for(MsgInfo info : infos){
            redisTemplate.opsForSet().add("msg",info);
        }
    }


    /**更改用户的状态*/
    @OnEvent(value = "setUserStatus")
    public void setUserStatus(SocketIOClient client, AckRequest request, String status,String content){

        String token = client.getHandshakeData().getSingleUrlParam("token");
        String fromUser = JWTUtil.getUsername(token);

        /**缓存操作者的操作时间*/
        redisTemplate.opsForValue().set("time_"+fromUser+"",new Date());

        String sessionId =  redisTemplate.opsForValue().get(fromUser).toString();
        boolean exist = client.getSessionId().toString().equals(sessionId);

        if(exist){
            /**忙碌*/
            if(busyStatus.equals(status)) {
                /**删除在线用户缓存*/
                redisTemplate.opsForSet().remove("onlineUsers", fromUser);
                /**加入忙碌用户缓存*/
                redisTemplate.opsForSet().add("busyUsers", fromUser);
                /**将自动回复内容存入缓存*/
                redisTemplate.opsForValue().set("content_"+fromUser+"",content);
            }
            /**离线*/
            if(offlineStatus.equals(status)){
                if (request.isAckRequested()) {
                    request.sendAckData("200");
                    client.disconnect();
                }
            }
            /**在线*/
            if(onlineStatus.equals(status)){
                /**删除在线用户缓存*/
                redisTemplate.opsForSet().remove("busyUsers", fromUser);
                /**加入忙碌用户缓存*/
                redisTemplate.opsForSet().add("onlineUsers", fromUser);
                /**将自动回复内容清空*/
                redisTemplate.delete("content_"+fromUser+"");
            }
            logger.info("更改用户状态成功！");
            resCode = 200;
        }else{
            logger.error("非法操作，不是认证的客户端！");
            resCode = 401;
        }

        if (request.isAckRequested()) {
            request.sendAckData(resCode);
        }

        logger.info("在线用户人数：" +  redisTemplate.opsForSet().members("onlineUsers"));
        logger.info("忙碌用户人数：" +  redisTemplate.opsForSet().members("busyUsers"));
        server.getBroadcastOperations().sendEvent("onlineUsers",  redisTemplate.opsForSet().members("onlineUsers"));
        server.getBroadcastOperations().sendEvent("busyUsers",  redisTemplate.opsForSet().members("busyUsers"));
    }




    /**离线消息接受**/
    @OnEvent(value = "getOfflineMsg")
    public void getOfflineMsg(SocketIOClient client, AckRequest request, Date dateTime) {

        String token = client.getHandshakeData().getSingleUrlParam("token");
        String fromUser = JWTUtil.getUsername(token);

        Date offlineDate = dateTime;
        /**有缓存的时间*/
        if(redisTemplate.hasKey("time_"+fromUser+"")){
            Date stringDate = (Date) redisTemplate.opsForValue().get("time_"+fromUser+"");
            try{
                Date redisDate = stringDate;
                if(redisDate.getTime() > dateTime.getTime()) {
                    offlineDate = redisDate;
                }
            }catch(Exception e){
                logger.error("日期格式转换出错！");
            }
        }

        String sessionId =  redisTemplate.opsForValue().get(fromUser).toString();
        /**校验是否是合法用户*/
        boolean exist = client.getSessionId().toString().equals(sessionId);
        if (exist) {

            long diff = System.currentTimeMillis()- offlineDate.getTime();
            boolean res = redisTemplate.hasKey("msg");
            /**当前用户的离线时间小于缓存入库时间，需要的数据在缓存中*/
            if( (diff<diffTime) && (redisTemplate.hasKey("msg")==true)){
                msgInfoRepository.save(redisTemplate.opsForSet().members("msg"));
                redisTemplate.delete("msg");
            }
            List<MsgInfo> list = msgInfoRepository.findMsgInfoByToUserAndDateGreaterThanEqual(fromUser,offlineDate);
            client.sendEvent("offlineMsg",new VoidAckCallback(timeOut) {
                @Override
                protected void onSuccess() {
                    logger.info("确认客户端收到了服务器下发的离线消息！");
                }
                @Override
                public void onTimeout() {
                    client.sendEvent("offlineMsg", list);
                }
            },list);
            resCode = 200;
        }else {
            logger.error("非法操作，不是认证的客户端！");
            resCode = 401;
        }

        if (request.isAckRequested()) {
            request.sendAckData(resCode);
        }
    }



    /**消息撤回*/
    @OnEvent(value = "callbackMsg")
    public void callbackMsg(SocketIOClient client, AckRequest request, MsgInfo data) {

        String token = client.getHandshakeData().getSingleUrlParam("token");
        String fromUser = JWTUtil.getUsername(token);

        int hasSend = 0;
        int error = 0;
        String sessionId =  redisTemplate.opsForValue().get(fromUser).toString();
        boolean exist = client.getSessionId().toString().equals(sessionId);

        if(exist){
            long dif = System.currentTimeMillis()-data.getDate().getTime();
            /** 超过设定的撤回时间限制*/
            if(dif>callbackTime){
                resCode = 400;
            }else{
                boolean res = redisTemplate.opsForSet().isMember("msg",data);
                try{
                    if(res){
                        redisTemplate.opsForSet().remove("msg",data);
                    }else{
                        MsgInfo info =  msgInfoRepository.findMsgInfoByDate(data.getDate());
                        msgInfoRepository.delete(info.getMsgId());
                    }
                }catch(Exception e){
                    client.sendEvent("exception","该条消息不存在，已删除！");
                    error=1;
                }

                if(error==0){
                    String toUser = data.getToUser();
                    if(redisTemplate.hasKey(toUser)){
                        UUID uid = UUID.fromString(redisTemplate.opsForValue().get(toUser).toString());
                        server.getClient(uid).sendEvent("callbackMsg",new VoidAckCallback(timeOut) {
                            @Override
                            protected void onSuccess() {
                                logger.info("确认客户端收到了服务器下发的撤销成功消息！");
                            }
                            @Override
                            public void onTimeout() {
                                server.getClient(uid).sendEvent("callbackMsg",data);
                            }
                        },data);
                    }
                    resCode = 200;
                }else{
                    if (request.isAckRequested()) {
                        request.sendAckData(500);
                    }
                    hasSend=1;
                }
            }
        }else{
            logger.error("非法操作，不是认证的客户端！");
            resCode = 401;
        }

        if (request.isAckRequested() && hasSend ==0) {
            request.sendAckData(resCode);
        }
    }
}

```

ScheduledTasks

```java
package com.njnsi.netty.handler;

import com.njnsi.netty.repository.MsgInfoRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

/**
 * ScheduledTasks class
 *
 * @author gqsu
 * @date 2018/04/05
 */
@Component
public class ScheduledTasks {

    private Logger logger = LoggerFactory.getLogger(ScheduledTasks.class);

    @Autowired
    private MsgInfoRepository msgInfoRepository;

    @Autowired
    private RedisTemplate redisTemplate;

    private int msgDelayCount = 1;

    private int roomMsgDelayCount = 1;

    @Scheduled(fixedDelay = 1000*60*10)        //表示当前方法执行完毕10分钟后，Spring scheduling会再次调用该方法
    public void msgInsert() {

        if(redisTemplate.hasKey("msg")){
            logger.info("第{}次执行方法msg", msgDelayCount++);
            msgInfoRepository.save(redisTemplate.opsForSet().members("msg"));
            redisTemplate.delete("msg");
        }

        if (redisTemplate.hasKey("roomMsg")) {
            logger.info("第{}次执行方法roomMsg", roomMsgDelayCount++);
            msgInfoRepository.save(redisTemplate.opsForSet().members("roomMsg"));
            redisTemplate.delete("roomMsg");
        }
    }
}

```

>对你所访问的服务端设置的IP与你的IP不对应，如我以上设置为localhost，只能本地测试，如果要其他人也能调用，请改成IP地址 调用。

### 问题总结

报错：Failed to load http://localhost:8081/socket.io/?EIO=3&transport=polling&t=1523265142408-2: The 'Access-Control-Allow-Origin' header has a value 'null' that is not equal to the supplied origin. 
Origin 'null' is therefore not allowed access.

>典型的跨域问题。跨域是指 不同域名之间相互访问。跨域，指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对JavaScript施加的安全限制。
也就是如果在A网站中，我们希望使用Ajax来获得B网站中的特定内容 
如果A网站与B网站不在同一个域中，那么就出现了跨域访问问题。

参考：https://blog.csdn.net/lenkvin/article/details/79482205



更多关于即时通讯的信息，请访问[即时通讯网](http://www.52im.net/)查看