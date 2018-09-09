---
layout: post
title: 基于apacheJames的WebMail收发第三方邮件
date: 2018-02-28 00:57:14
tags: Apache James
categories: Apache James
---


#### 表结构
```
james_mail    :邮件表。用于存储邮件信息
```

#### 说明

1、james_mailbox记录用户的收发邮件操作，User_Name是执行的用户名，MailBox_Name对应操作类别
INBOX（收取）和Sent（发送）。
2、


#### 修改james_mailbox表结构即可WebMail收取第三方邮件

新增NETBOX_PASSWORD（其他邮箱加密后的密码）、PASSWORD_HASH_ALGORITHM（加密所使用的算法）、
NEWEST_SENT_DATE（最新的收取邮件时间）三个字段

参考文献：[文章](http://www.doc88.com/p-8582142643394.html)