---
layout: post
title: elasticsearch 6.x的基本概念与安装
date: 2018-03-14 10:49:15
tags: elasticsearch
categories: elasticsearch
---

### 基本知识

#### 简介
ElasticSearch（简称ES）是一个分布式、Restful的搜索及分析服务器，设计用于分布式计算；能够达到实时搜索，稳定，可靠，快速。和Apache Solr一样，它也是基于Lucence的索引服务器，

而ElasticSearch对比Solr的优点在于：

- 轻量级：安装启动方便，下载文件之后一条命令就可以启动。
- Schema free：可以向服务器提交任意结构的JSON对象，Solr中使用schema.xml指定了索引结构。
- 多索引文件支持：使用不同的index参数就能创建另一个索引文件，Solr中需要另行配置。
- 分布式：Solr Cloud的配置比较复杂。

<!-- more -->

2013年初，GitHub抛弃了Solr，采取ElasticSearch 来做PB级的搜索。

近年ElasticSearch发展迅猛，已经超越了其最初的纯搜索引擎的角色，现在已经增加了数据聚合分析（aggregation）和可视化的特性，

如果你有数百万的文档需要通过关键词进行定位时，ElasticSearch肯定是最佳选择。当然，如果你的文档是JSON的，你也可以把ElasticSearch当作一种“NoSQL数据库”， 应用ElasticSearch数据聚合分析（aggregation）的特性，针对数据进行多维度的分析。

ElasticSearch一些国内外的优秀案例：

Github：“GitHub使用ElasticSearch搜索20TB的数据，包括13亿文件和1300亿行代码”。

SoundCloud：“SoundCloud使用ElasticSearch为1.8亿用户提供即时而精准的音乐搜索服务”。

百度：百度目前广泛使用ElasticSearch作为文本数据分析，采集百度所有服务器上的各类指标数据及用户自定义数据，通过对各种数据进行多维分析展示，辅助定位分析实例异常或业务层面异常。目前覆盖百度内部20多个业务线（包括casio、云分析、网盟、预测、文库、直达号、钱包、风控等），单集群最大100台机器，200个ES节点，每天导入30TB+数据。

#### 概念

- Cluster 和 Node

ES可以以单点或者集群方式运行，以一个整体对外提供search服务的所有节点组成cluster，组成这个cluster的各个节点叫做node。

- Index

这是ES存储数据的地方，类似于关系数据库的database。

- Shards

索引分片，这是ES提供分布式搜索的基础，其含义为将一个完整的index分成若干部分存储在相同或不同的节点上，这些组成index的部分就叫做shard。

- Replicas

索引副本，ES可以设置多个索引的副本，副本的作用一是提高系统的容错性，当个某个节点某个分片损坏或丢失时可以从副本中恢复。二是提高ES的查询效率，ES会自动对搜索请求进行负载均衡。

- Recovery

代表数据恢复或叫数据重新分布，ES在有节点加入或退出时会根据机器的负载对索引分片进行重新分配，挂掉的节点重新启动时也会进行数据恢复。

- Gateway

ES索引快照的存储方式，ES默认是先把索引存放到内存中，当内存满了时再持久化到本地硬盘。gateway对索引快照进行存储，当这个ES集群关闭再重新启动时就会从gateway中读取索引备份数据。

- Discovery.zen

代表ES的自动发现节点机制，ES是一个基于p2p的系统，它先通过广播寻找存在的节点，再通过多播协议来进行节点之间的通信，同时也支持点对点的交互。

- NTransport

代表ES内部节点或集群与客户端的交互方式，默认内部是使用tcp协议进行交互，同时它支持http协议（json格式）。



### 安装elasticsearch


Elastic Search是一个开源的，分布式，实时搜索和分析引擎。ES是一个基于Lucene的分布式全文搜索服务器，和SQL Server的全文索引（Fulltext Index）有点类似，
都是基于分词和分段的全文搜索引擎，具有分词，同义词，词干查询的功能，但是ES天生具有分布式和实时的属性


按照官方文档的内容进行操作：[官网](https://www.elastic.co/downloads/elasticsearch)

运行[http://localhost:9200](http://localhost:9200) 查看是否启动成功

>注意：安装目录不能包含中文和空格，如果含有空格需要用括号将其括起来。

可选择根据需要选择相关配置：修改安装目下 ~/config/elasticsearch.yml文件， 相关配置参数：

```yml
##集群名称
cluster.name: elasticsearch

##节点名称
node.name: "node1"

##节点是否存储数据
node.data: true

##索引分片数
index.number_of_shards: 5

##索引副本数
index.number_of_replicas: 1

##数据目录存放位置
path.data: /data/elasticsearch/data

##日志数据存放位置
path.logs: /data/elasticsearch/log

##索引缓存
index.cache.field.max_size: 500000

##索引缓存过期时间
index.cache.field.expire: 5m
```


安装x-pack，地址与详细教程： [参考官网](https://www.elastic.co/downloads/x-pack)

>注意：执行设置自动密码命令`bin/x-pack/setup-passwords auto`，一定要先启动elasticsearch服务，然后用生成的密码去登陆。
如果想自定义密码，用命令：`bin/x-pack/setup-passwords interactive`，按照提示完成即可。


### 安装Kibana插件


Kibana插件可做近实时系统数据分析，每个分片的刷新频率为1s一次

下载地址：https://www.elastic.co/downloads/kibana

>注意，kibana 的版本必须与 ElasticSearch 的版本对应

解压缩后，修改../config下的配置文件 kibana.yml

```properties
server.host: 127.0.0.1
elasticsearch.url: "http://127.0.0.1:9200"
```

然后，浏览器直接访问 http://localhost:5601。
![](http://p2jr3pegk.bkt.clouddn.com/es01-1.png)


注意：未安装x-pack插件的kibana只具有基本功能，Monitoring、Graph等功能不能使用。

安装x-pack，地址与详细教程： https://www.elastic.co/guide/en/kibana/6.2/installing-xpack-kb.html

在线安装：进入kibana目录，运行`bin/kibana-plugin install x-pack`（linux用户注意不要使用root用户，可使用`sudo -u kibana bin/kibana-plugin install x-pack`）

>若是在线安装出现问题，则需要手动安装，下载地址为：`https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.2.2.zip`,
下载完成后，运行将后面的path路径指明为你的下载包存放路径即可：`bin/kibana-plugin install file:///path/to/file/x-pack-6.2.3.zip`

修改../config下的配置文件 kibana.yml

```properties
elasticsearch.username: "kibana"
elasticsearch.password: "kibanapassword"
```

这里关于 elastic、kibana和logstash用户的区别可参见：[官方文档](https://www.elastic.co/guide/en/kibana/6.2/settings-xpack-kb.htmlyml)


### 安装ik分词


下载地址为： https://github.com/medcl/elasticsearch-analysis-ik

>注意，ik 分词的版本必须与 ElasticSearch 的版本对应。下载下来的为源代码，运行`mvn package`得到打包文件，并做一些其他操作。


推荐直接使用编译好的文件，下载： https://github.com/medcl/elasticsearch-analysis-ik/releases
解压缩后，更改最里层目录文件夹的名称为`analysis-ik`，并拷贝其至`elasticsearch-6.2.2/plugins`目录下

运行`elasticsearch-6.2.2/bin`下的 `elasticsearch.bat`，启动elasticsearch。

测试基本功能：

```json
GET _analyze?pretty
{
  "analyzer": "ik_smart",   //ik_smart和ik_max_word两个字段可选
  "text":"安徽省长江流域"
}
```

返回：

```json
{
  "tokens": [
    {
      "token": "安徽省",
      "start_offset": 0,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "长江流域",
      "start_offset": 3,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 1
    }
  ]
}
```

>默认使用的是标准分词器(standard)。扩展：使用自己的分词字典。

>这里需要特别指出的是，英文分词(english)会将的过去分词、现在分词、复数等形式的单词转换为最基本的形态。

下载的analysis-ik都是别人定义好的一些已有字典。可新建new_word.dic，在里面定义自己的一些规范即可。

最后别忘了更改配置文件 IKAnalyzer.cfg.xml ，然后重启即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>IK Analyzer 扩展配置</comment>
    <!--用户可以在这里配置自己的扩展字典 -->
    <entry key="ext_dict">custom/new_word.dic</entry>
     <!--用户可以在这里配置自己的扩展停止词字典-->
    <entry key="ext_stopwords"></entry>
    <!--用户可以在这里配置远程扩展字典 -->
    <!-- <entry key="remote_ext_dict">words_location</entry> -->
    <!--用户可以在这里配置远程扩展停止词字典-->
    <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

### 安装Logstash

>主要用于构建 ELK 服务，即安装 elasticsearch + Logstash + Kibana

去官网下载：https://www.elastic.co/downloads/logstash

下载完成后，解压

```shell
cd logstash-6.2.2
bin/logstash.bat -e 'input { stdin { } } output { stdout {} }'
```

以上是正常的控制台写入输出模式。一定要注意`bin/logstash.bat`运行，不能切换到`bin`目录在运行。

按照官方建议，在当前目录下，新建 logstash.conf 文件加入配置：

这里以导入数据为例：

```sh
  file {
    path => "F:/ElasticSearch/logstash-5.4.1/shakespeare.json/"       ##数据源文件
    start_position => "beginning"
    ignore_older => 0
    codec => "json"
#    sincedb_path => "/dev/null"
    type => "wjb_log"
  }
}

filter{
    if [bboy_id] {
        mutate { add_tag => "BBOY" }
    }
}

output {
    if [bboy_id] {
        elasticsearch {
            hosts => "localhost:9200"
            index => "bboy-%{+YYYY.MM.dd}"
            document_type => "bboy_log"
        }
    } else {
        elasticsearch {
            hosts => "localhost:9200"
            index => "wjb-%{+YYYY.MM.dd}"
            document_type => "wjb_log"
        }
    }
}
```

接着，只需运行配置文件即可。可通过 kibana 查看数据是否导入成功。

```powershell
$ bin/logstash.bat -f logstash.conf
```

完整的配置文件说明如下：

```sh
#整个配置文件分为三部分：input,filter,output
#参考这里的介绍 https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html
input {
  #file可以多次使用，也可以只写一个file而设置它的path属性配置多个文件实现多文件监控
  file {
    #type是给结果增加了一个属性叫type值为"<xxx>"的条目。这里的type，对应了ES中index中的type，即如果输入ES时，没有指定type，那么这里的type将作为ES中index的type。
    type => "apache-access" 
    path => "/apphome/ptc/Windchill_10.0/Apache/logs/access_log*"
    #start_position可以设置为beginning或者end，beginning表示从头开始读取文件，end表示读取最新的，这个也要和ignore_older一起使用。
    start_position => beginning
    #sincedb_path表示文件读取进度的记录，每行表示一个文件，每行有两个数字，第一个表示文件的inode，第二个表示文件读取到的位置（byteoffset）。默认为$HOME/.sincedb*
    sincedb_path => "/opt/logstash-2.3.1/sincedb_path/access_progress"
    #ignore_older表示了针对多久的文件进行监控，默认一天，单位为秒，可以自己定制，比如默认只读取一天内被修改的文件。
    ignore_older => 604800
    #add_field增加属性。这里使用了${HOSTNAME}，即本机的环境变量，如果要使用本机的环境变量，那么需要在启动命令上加--alow-env。
    add_field => {"log_hostname"=>"${HOSTNAME}"}
    #这个值默认是\n 换行符，如果设置为空""，那么后果是每个字符代表一个event
    delimiter => ""
    #这个表示关闭超过（默认）3600秒后追踪文件。这个对于multiline来说特别有用。... 这个参数和logstash对文件的读取方式有关，两种方式read tail，如果是read
    close_older => 3600
    coodec => multiline {
      pattern => "^\s"
      #这个negate是否定的意思，意思跟pattern相反，也就是不满足patter的意思。
#      negate => ""
      #what有两个值可选 previous和next，举例说明，java的异常从第二行以空格开始，这里就可以pattern匹配空格开始，what设置为previous意思是空格开头这行跟上一行属于同一event。另一个例子，有时候一条命令太长，当以\结尾时表示这行属于跟下一行属于同一event，这时需要使用negate=>true，what=>'next'。
      what => "previous"
      auto_flush_interval => 60
    }
  }
  file { 
    type => "methodserver-log" 
    path => "/apphome/ptc/Windchill_10.0/Windchill/logs/MethodServer-1604221021-32380.log" 
    start_position => beginning 
    sincedb_path => "/opt/logstash-2.3.1/sincedb_path/methodserver_process"
#    ignore_older => 604800
  }
}
filter{
  #执行ruby程序，下面例子是将日期转化为字符串赋予daytag
  ruby {
    code => "event['daytag'] = event.timestamp.time.localtime.strftime('%Y-%m-%d')"
  }
  # if [path] =~ "access" {} else if [path] =~ "methodserver" {} else if [path] =~ "servermanager" {} else {} 注意语句结构
  if [path] =~ "MethodServer" { #z这里的=~是匹配正则表达式
    grok {
      patterns_dir => ["/opt/logstash-2.3.1/patterns"] #自定义正则匹配
#      Tue 4/12/16 14:24:17: TP-Processor2: hirecode---->77LS
      match => { "message" => "%{DAY:log_weekday} %{DATE_US:log_date} %{TIME:log_time}: %{GREEDYDATA:log_data}"}
    }
    #mutage是做转换用的
    mutate { 
      replace => { "type" => "apache" } #替换属性值
      convert => { #类型转换
        "bytes" => "integer" #例如还有float
        "duration" => "integer"
        "state" => "integer"
      }
    #date主要是用来处理文件内容中的日期的。内容中读取的是字符串，通过date将它转换为@timestamp。参考https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html#plugins-filters-date-match
#    date {
#      match => [ "logTime" , "dd/MMM/yyyy:HH:mm:ss Z" ]
#    }
  }else if [type] in ['tbg_qas','mbg_pre'] { # if ... else if ... else if ... else结构
  }else {
    drop{} # 将event丢弃
  }
}
output {
  stdout{ codec=>rubydebug} # 直接输出，调试用起来方便
  # 输出到redis
  redis {
    host => '10.120.20.208'
    data_type => 'list'
    key => '10.99.201.34:access_log_2016-04'
  }
  # 输出到ES
  elasticsearch {
    hosts =>"192.168.0.15:9200"
    index => "%{sysid}_%{type}"
    document_type => "%{daytag}"
  }
}
```

使用参考1：http://blog.51cto.com/tchuairen/1840596

使用参考2：https://blog.csdn.net/iguyue/article/details/77006201
