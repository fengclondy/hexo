---
layout: post
title: ES之从mysql中获取数据导入至ES
date: 2018-07-27 02:12:00
tags: elasticsearch
categories: elasticsearch
---

### elasticsearch-jdbc

官网下载地址：[官网](https://www.elastic.co/downloads/jdbc-client)


#### 方式一，手动编写配置文件操作
1、安装elasticsearch-jdbc中间键
```powershell
wget http://xbib.org/repository/org/xbib/elasticsearch/importer/elasticsearch-jdbc/2.3.4.0/elasticsearch-jdbc-2.3.4.0-dist.zip 
unzip elasticsearch-jdbc-2.3.4.0-dist.zip 
```

2、安装dos2unix

```powershell
yum install dos2unix
```

<!-- more -->

3、编写配置文件mysql_accountinfo.sh

```sh
#!/bin/sh

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
bin=${DIR}/../bin
lib=${DIR}/../lib

echo '{
    "type":"jdbc",
    "jdbc":{
        "url":"jdbc:mysql://localhost:3306/kakme",
        "user":"root",
        "password":"123456",
        "sql":"select id as _id , account_name as accountName , nick_name nickName from account_info",
        "elasticsearch" : {
            "cluster" : "nini",
            "host" : "localhost",
            "port" : 9300
        },
        "index":"cwenao",
        "type":"accountinfo",

        "type_mapping" :{
            "account_info": {
                "properties": {
                    "id":{
                        "type":"string",
                        "index":"not_analyzed"
                    },
                    "accountName":{
                        "type":"string"
                    },
                    "nickName":{
                        "type":"string"
                    }
                }
            }
        }
    }
}' | java \
    -cp "${lib}/*" \
    -Dlog4j.configurationFile=${bin}/log4j2.xml \
    org.xbib.tools.Runner \
    org.xbib.tools.JDBCImporter
```

4、上传服务器并用dos2unix转换
```shell
dos2unix mysql_accountinfo.sh
chmod 700 mysql_accountinfo.sh
sh mysql_accountinfo.sh
```

5、查看结果

```
curl -XGET 'http://localhost:9200/cwenao/accountinfo/_search?pretty'
```


参考博客：https://blog.csdn.net/cwenao/article/details/54944937

#### 其他方式

参考中文连接：[中文社区](https://elasticsearch.cn/article/687#0-tsina-1-2124-397232819ff9a47a7b7e80a40613cfe1)

或者使用logstash，参考连接：[连接](https://elastic-search-in-action.medcl.com/3.6_data_migration.html)


参考：https://doc.yonyoucloud.com/doc/logstash-best-practice-cn/get_start/hello_world.html


### Logstash导入Mysql数据

#### 安装

1、去[官网](https://www.elastic.co/downloads/logstash)下载zip安装包

2、下载JDBC链接驱动，选择java源，下载地址：[地址](https://dev.mysql.com/downloads/connector/j/)

#### 使用

1、在 logstash 解压后的`/bin`目录下，创建一个 Logstash 的配置文件：jdbc.conf。

```sh
input {
  jdbc {
    jdbc_driver_library => "F:\mysql-connector-java-5.1.38\mysql-connector-java-5.1.38\mysql-connector-java-5.1.38-bin.jar"
    jdbc_driver_class => "Java::com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/im"
    jdbc_user => "root"
    jdbc_password => "root"
    statement => "SELECT `msg_info`.`msgId`, `msg_info`.`content`, `msg_info`.`date`, `msg_info`.`fromUser`, `msg_info`.`toUser`, `msg_info`.`msgType`,`msg_info`.`hasRead`  FROM `im`.`msg_info`"
    jdbc_paging_enabled => "true"
    jdbc_page_size => "50000"
  }
}

filter {
}

output {
  stdout {
    codec => rubydebug
  }
 
  elasticsearch {
    index => "forum-mysql"
  }
}
```

>这里需要注意的是，JDBC驱动不要安装最新版的8.x，启动不了，暂时还不知道为什么，选用的是5.X版本。

2、执行命令，启动 Logstash
```powershell
$ cd ~/logstash-6.3.0/bin
$ logstash -f jdbc.conf
```

>如果报错，可能是SQL表中存在为null的字段，删除即可。

3、登录kibana查看数据：
```
GET forum-mysql/_search
```
![](http://p2jr3pegk.bkt.clouddn.com/logstash01-1.png)

#### 报错修复

`Error: Java::com.mysql.jdbc.Driver not loaded. Are you sure you've included the correct jdbc driver in :jdbc_driver_library?`

--> 请尝试使用 JDK 8 来运行 Logstash。
