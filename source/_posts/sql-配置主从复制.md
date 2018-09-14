---
layout: post
title: sql01
date: 2018-08-08 08:11:24
tags: sql
categories: sql
---

### MYSQL主从复制
>准备工作：- 1.版本一致  - 2.初始化表，并在后台启动mysql - 3.修改root的密码


1、修改主服务器master：
```shell
#vi /etc/my.cnf
[mysqld]
log-bin=mysql-bin   //[必须]启用二进制日志
server-id=232      //[必须]服务器唯一ID，默认是1，一般取IP最后一段
```

2、修改从服务salve:
```shell
#vi /etc/my.cnf
[mysqld]
log-bin=mysql-bin   //[不是必须]启用二进制日志
server-id=222      //[必须]服务器唯一ID，默认是1，一般取IP最后一段
```

3、重启两台服务器的mysql:
```shell
service mysqld restart

//若启动不成功,查看日志，一般是my.cnf配置问题
cat /var/log/mysqld.log
```

<!-- more -->

4、在主服务器上建立帐户并授权slave:

```shell
GRANT REPLICATION SLAVE ON *.* to 'hs'@'%' identified by 'a123.+-'; 
//一般不用root帐号，@;%;表示所有客户端都可能连，只要帐号(hs)，密码正确(a123.+-)，此处可用具体客户端IP代替，如192.168.0.1，加强安全。
```

5、登录主服务器的mysql，查询master的状态
```mysql
  mysql>show master status;
   +------------------+----------+--------------+------------------+
   | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
   +------------------+----------+--------------+------------------+
   | mysql-bin.000003 |      712 |              |                  |
   +------------------+----------+--------------+------------------+
   1 row in set (0.00 sec)
  // 注：执行完此步骤后不要再操作主服务器MYSQL，防止主服务器状态值变化
```

6、配置从服务器Slave：
```mysql
mysql>change master to master_host='192.168.0.232',master_user='hs',master_password='a123.+-',master_log_file='mysql-bin.000003',master_log_pos=712;   
Mysql>start slave;    //启动从服务器复制功能,注意上面的用户名，密码，端口等
```

7、检查从服务器复制功能状态：
```mysql
mysql> show slave status\G
  Slave_IO_State: Waiting for master to send event
  Master_Host: 192.168.0.232  //主服务器地址
  Master_User: hs   //授权帐户名，尽量避免使用root
  Master_Port: 3306    //数据库端口，部分版本没有此行
  Connect_Retry: 60
  Master_Log_File: mysql-bin.000003
  Read_Master_Log_Pos: 600     //#同步读取二进制日志的位置，大于等于Exec_Master_Log_Pos
  Relay_Log_File: ddte-relay-bin.000003
  Relay_Log_Pos: 251
  Relay_Master_Log_File: mysql-bin.000004
  Slave_IO_Running: Yes    //此状态必须YES
  Slave_SQL_Running: Yes     //此状态必须YES
                    ......

//注：Slave_IO及Slave_SQL进程必须正常运行，即YES状态，否则都是错误的状态(如：其中一个NO均属错误)。
```
主从服务器配置完成后，建立一个库，插入数据进行测试是否正常。