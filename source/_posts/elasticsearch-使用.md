---
layout: post
title: ES 6.x使用
date: 2018-03-15 05:57:30
tags: elasticsearch
categories: elasticsearch
---

ElasticSearch 是一个基于 Lucene 的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于 RESTful web 接口。
Elasticsearch 是用 Java 开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。
设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。


>6.x版本与前几个版本的区别：[重大变化](https://www.felayman.com/articles/2017/11/15/1510733125940.html?utm_medium=hao.caibaojian.com&utm_source=hao.caibaojian.com)


### 基本概念

1、索引

索引(index)是ElasticSearch存放具体数据的地方，是一类具有相似特征的文档的集合。
ElasticSearch中索引的概念具有不同意思，这里的索引相当于关系数据库中的一个数据库实例
在ElasticSearch中索引还可以作为动词，表示对数据进行索引操作。

2、类型

索引(index)是ElasticSearch存放具体数据的地方，是一类具有相似特征的文档的集合。
ElasticSearch中索引的概念具有不同意思，这里的索引相当于关系数据库中的一个数据库实例。
在ElasticSearch中索引还可以作为动词，表示对数据进行索引操作。

3、文档

文档是ElasticSearch可被索引的基础逻辑单元，相当于关系数据库中数据表的一行数据。
ElasticSearch的文档具有JSON格式，由多个字段组成，字段相当于关系数据库中列的概念。

4、分片

当数据量较大时，索引的存储空间需求超出单个节点磁盘容量的限制，或者出现单个节点处理速度较慢。
为了解决这些问题，ElasticSearch将索引中的数据进行切分成多个分片（shard），每个分片存储这个索引的一部分数据，分布在不同节点上。
当需要查询索引时，ElasticSearch将查询发送到每个相关分片，之后将查询结果合并，这个过程对ElasticSearch应用来说是透明的，用户感知不到分片的存在。 
一个索引的分片一定指定，不再修改。

5、副本

分片全称是主分片，简称为分片。主分片是相对于副本来说的，副本是对主分片的一个或多个复制版本（或称拷贝），
这些复制版本（拷贝）可以称为复制分片，可以直接称之为副本。
当主分片丢失时，集群可以将一个副本升级为新的主分片。

<!-- more -->

### 对比

|ElasticSearch	|RDBMS|
| ------------- |:-----:|
|索引（index）	 |数据库（database）|
|类型（type）	 |表（table）|
|文档（document）	|行（row）|
|字段（field）	 |列（column）|
|映射（mapping）	|表结构（schema）|
|全文索引	     |索引|
|查询DSL	      |SQL|
|GET	      |select|
|PUT/POST	  |update|
|DELETE	      |delete|


### 一些基本的操作

>此处可使用curl、postman或者kibana插件

### 索引

#### 创建索引

```
PUT test
```

返回结果：

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "test"
}
```

注意：索引名不能包含大些字母、不能重复创建  
但可指定参数，例子：

```
PUT blog
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  } 
}
```

#### 查看索引

查看索引全部信息：

```
GET blog
```

查看配置信息

```
GET blog/_settings
```

返回：

```
{
  "blog": {
    "settings": {
      "index": {
        "creation_date": "1515458969949",
        "number_of_shards": "3",
        "number_of_replicas": "1",
        "uuid": "A7pKNO7bTgucu1uNgmXlQg",
        "version": {
          "created": "5060399"
        },
        "provided_name": "blog"
      }
    }
  }
}
```

查看多个索引配置：

```
GET test,blog/_settings
```

#### 删除索引

```
DELETE test
```

返回：

```
{
  "acknowledged": true
}
```

#### 索引的打开与关闭

关闭

```
POST blog/_close
```

打开

```
POST blog/_open
```

### 文档

#### 新建文档

```
PUT blog/csdn/1
{
  "id":1,
  "title":"Elasticsearch简介",
  "author":"chengyuqiang",
  "content":"Elasticsearch是一个基于Lucene的搜索引擎"
}
```

返回：

```
{
  "_index": "blog",    //索引名
  "_type": "csdn",    //类型名
  "_id": "1",    //文档ID
  "_version": 1,    //版本号
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "created": true
}
```

在录入两条数据

```
POST blog/csdn/2
{
  "id":2,
  "title":"Git简介",
  "author":"chengyuqiang",
  "content":"Git是一个版本控制软件"
}

POST blog/csdn
{
  "id":3,
  "title":"Java编程",
  "author":"chengyuqiang",
  "content":"Java面向对象程序设计"
}
```

>注明： 如果未指定文档的ID，elasticsearch会自动给其赋值一个随机值。
如果指定的ID已存在，则默认执行的是更新操作，文档数据会被替换为现有的，版本号则加1。

当然，如果任想自定义id，但是又不想覆盖之前的数据，可以加入`_create`参数：

```
POST blog/csdn/4/_create
```

若创建成功，返回 201 Created 的 HTTP 响应码；若创建失败，将会返回 409 Conflict 响应码，以及对应的错误信息。


#### 获取文档

```
GET blog/csdn/1
```

返回：

```
{
  "_index": "blog",
  "_type": "csdn",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "id": 1,
    "title": "Elasticsearch简介",
    "author": "chengyuqiang",
    "content": "Elasticsearch是一个基于Lucene的搜索引擎"
  }
}
```

>默认返回10条以内数据  

批量获取数据：

```
GET blog/csdn/_mget
{
  "ids":["1","2"]
}
```

返回：

```
{
  "docs": [
    {
      "_index": "blog",
      "_type": "csdn",
      "_id": "1",
      "_version": 1,
      "found": true,
      "_source": {
        "id": 1,
        "title": "Elasticsearch简介1",
        "author": "chengyuqiang",
        "content": "Elasticsearch是一个基于Lucene的搜索引擎"
      }
    }，
    {
      "_index": "blog",
      "_type": "csdn",
      "_id": "2",
      "_version": 1,
      "found": true,
      "_source": {
        "id": 1,
        "title": "Elasticsearch简介2",
        "author": "gqsu",
        "content": "Elasticsearch"
      }
    }，
  ]
}
```

#### 更新文档

##### 1.更新数据 (创建的时候已经说明，这里不举例)

>注意：  

版本加1。 (每操作一次都会加1)
created标识为 false，因为同索引同类型下已经存在同ID的文档。  
在ES内部，_version为1的文件已经被标记“删除”，并添加了一个完整的新文档。  
旧文档不会立即消失，但是不能再访问它。

##### 2.更新字段  

```
POST blog/csdn/2/_update
{
  "script": {
    "source": "ctx._source.content=\"Git是一个开源的分布式版本控制软件\""  
  } 
}
```

##### 3.添加新字段

>使用`script`或者`doc`参数

```
POST blog/csdn/1/_update
{
  "script": "ctx._source.posttime=\"2018-01-09\""
}
```

或者采用下面这种形式：

```
POST blog/csdn/1/_update
{
   "doc" : {                        
      "posttime": "2018-01-09"
   }
}
```

返回结果：

```
{
  "_index": "blog",
  "_type": "csdn",
  "_id": "1",
  "_version": 4,
  "result": "updated",
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 14,
  "_primary_term": 9
}
```

>这里的方式1采用了脚本编程。拓展的 Groovy 脚本编程，可自行研究。

如果是为所有索引下的类添加字段：
```
PUT blog/csdn/_mapping
{
  "properties": {
    "message": {
      "type": "text",
      "fields": {
        "keyword": {
        "type": "keyword",
        "ignore_above": 256
        }
      }
    }
  }
}
```
>只要添加的字段名一样，属性会覆盖配置；如果不同，则会新增。但是新增并不会影响当前数据，当前数据的该字段仍为空。


#### 删除文档

```
DELETE blog/csdn/1 
```

返回：

```
{
  "_index": "blog",
  "_type": "csdn",
  "_id": "1",
  "_version": 2,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 6,
  "_primary_term": 5
}
```

>删除文档不会立即将文档从磁盘中删除，只是将文档标记为已删除状态。只要是操作一次，version的值都会加1，无论成功与否。
随着你不断的索引更多的数据，Elasticsearch 将会在后台清理标记为已删除的文档。


#### 文档搜索

##### 1.检索全部文档：

```
GET blog/_search
```

返回：

```
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "blog",
        "_type": "csdn",
        "_id": "2",
        "_score": 1,
        "_source": {
          "id": 2,
          "title": "Git简介",
          "author": "chengyuqiang",
          "content": "Git是一个版本控制软件"
        }
      },
      {
        "_index": "blog",
        "_type": "csdn",
        "_id": "1",
        "_score": 1,
        "_source": {
          "id": 1,
          "title": "Elasticsearch简介",
          "author": "chengyuqiang",
          "content": "Elasticsearch是一个基于Lucene的搜索引擎"
        }
      },
      {
        "_index": "blog",
        "_type": "csdn",
        "_id": "wkv472AB5R2olyYk97rN",
        "_score": 1,
        "_source": {
          "id": 3,
          "title": "Java编程",
          "author": "chengyuqiang",
          "content": "Java面向对象程序设计"
        }
      }
    ]
  }
}
```


##### 2.term查询

term查询用于查找指定字段中包含指定分词的文件，只有当查询分词和文档中的分词精确匹配时才被检索到。

```
GET blog/_search
{
  "query": {
    "term": {
      "title": "程"
    }
  }
}
```

未使用ik分词的查询结果(每个汉字被看做独立的一个词)：

```
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "blog",
        "_type": "csdn",
        "_id": "wkv472AB5R2olyYk97rN",
        "_score": 0.2876821,
        "_source": {
          "id": 3,
          "title": "Java编程",
          "author": "chengyuqiang",
          "content": "Java面向对象程序设计"
        }
      }
    ]
  }
}
```

>term查询有点类似于keyword关键字查询，形如"title.keyword"

当然，也可以完成同一个关键字多个词的查找：

```
GET blog/_search
{
  "query": {
    "terms": {
      "title": ["java","git"]
    }
  }
}
```

##### 3.match查询

与term精确查询不同，对于match查询，只要被查询字段中存在任何一个词项被匹配，就会搜索到该文档。

可参考上面的查询

```
## 方式1
GET blog/_search
{
  "query": {
    "match": {
      "title": {
        "query": "程序"
      }
    }
  }
  //,
  //"from": 1,     //位移
  //"size": 1     //数量
  //"explain": true   //分值排序解释
  //"sort"：[{"date":{"order":"desc"}}]   //排序规则，默认为_score,加上其他字段后score就全部为null值了
}

## 方式2
GET blog/_search
{
  "query": {
    "match": {
      "title": "程序"
    }
  }
}

## 方式3
GET blog/_search
{
  "query": {
    "match_phrase": {
      "title": "程序"
    }
  }
}
```

>此处需要特别注意的是我们查询的是title中的“程序”，但是返回的结果里并不包含“程序”的全部。
原因是程序会自动为title赋值成默认的standard分词，将程序会分为“程”和“序”两部分，查询时只要匹配任一即返回了。
在深入分析理解里会再次提到，此处做了解。若想完全匹配，则可指定为“title.keyword”，或者使用短语匹配“match_phrase”。


返回：

```
{
  "took": 37,                 //该操作的耗时（单位为毫秒）
  "timed_out": false,            //是否超时
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,                      //记录数
    "max_score": 0.2876821,       //最高的匹配程度
    "hits": [                  //命中的记录
      {
        "_index": "blog",
        "_type": "csdn",
        "_id": "wkv472AB5R2olyYk97rN",
        "_score": 0.2876821,             //匹配的程度，默认是按此字段降序排列
        "_source": {
          "id": 3,
          "title": "Java编程",
          "author": "chengyuqiang",
          "content": "Java面向对象程序设计"
        }
      }
    ]
  }
}
```
>>>>>>下面的方式可能不能用，新版不支持
稍微复杂一点的可以使用filter进行过滤:

```
GET blog/_search
{
  "query": {
    "match": {
      "title": {
        "query": "程序"
      }
    }
  },
  "filter": {
     ...
     }
  }
}
```

##### 4.短语搜索
```
GET blog/_search
{
    "query" : {
        "match_phrase" : {
            "content" : "Lucene"
        }
    }
}
```

##### 5.其他逻辑运算  

多个搜索关键字的or运算：

```
GET blog/_search
{
  "query" : { "match" : { "title" : "编程 简介" }}
}
```

多个关键词的and搜索：

```
GET blog/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "编程" } },
        { "match": { "title": "JAVA" } }
      ]
    }
  }
}
```



#### 其他

##### 只选取一个文档的某些字段（某一部分）
```
GET blog/csdn/1?_source=title,author
```

##### 检查文档是否存在
```
HEAD /my_store/products/2
```
>若文档存在， Elasticsearch 将返回一个 200 ok 的状态码；不存在则返回一个 404 - Not Found 的状态码。

##### 取回多个文档
```
GET /_mget
{
   "docs" : [               #固定参数
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "blog",
         "_type" :  "csdn",
         "_id" :    1
      }
   ]
}
```

返回结果:

```
{
  "docs": [
    {
      "_index": "website",
      "_type": "blog",
      "_id": "2",
      "_version": 1,
      "found": true,
      "_source": {
        "title": "watchman源码编译",
        "author": "李四",
        "postdate": "2016-12-23",
        "abstract": "CentOS7.x的watchman源码编译",
        "url": "https://url.cn/53844169"
      }
    },
    {
      "_index": "blog",
      "_type": "csdn",
      "_id": "1",
      "_version": 14,
      "found": true,
      "_source": {
        "id": 1,
        "title": "Elasticsearch简介",
        "author": "chengyuqiang",
        "content": "Elasticsearch是一个基于Lucene的搜索引擎",
        "posttime": "2018-01-09",
        "updateTime": "2018-03-09"
      }
    }
  ]
}
```

如果 index 和 type 都相同，则可以使用下面这种形式：

```
GET /blog/csdn/_mget
{
   "ids" : [ "1", "100" ]
}
```

返回：

```
{
  "docs": [
    {
      "_index": "blog",
      "_type": "csdn",
      "_id": "1",
      "_version": 14,
      "found": true,
      "_source": {
        "id": 1,
        "title": "Elasticsearch简介",
        "author": "chengyuqiang",
        "content": "Elasticsearch是一个基于Lucene的搜索引擎",
        "posttime": "2018-01-09",
        "updateTime": "2018-03-09"
      }
    },
    {
      "_index": "blog",
      "_type": "csdn",
      "_id": "100",
      "found": false
    }
  ]
}
```


### 路由

当索引一个文档的时候，文档会被存储到一个主分片中。Elasticsearch 根据公式：

`shard = hash(routing) % number_of_primary_shards`,即 文档id的hash函数值除以主分片的总数，

然后取余数，求出文档所在分片的位置进行存储。

显式指明路由位置的分片id:

```
PUT my_index/_doc/1?routing=user1&refresh=true 
{
  "title": "This is a document"
}
```


### 多索引多类型

`/_search`在所有的索引中搜索所有的类型  

`/gb/_search`在 gb 索引中搜索所有的类型  

`/gb,us/_search`在 gb 和 us 索引中搜索所有的文档  

`/g*,u*/_search`在任何以 g 或者 u 开头的索引中搜索所有的类型  

`/gb/user/_search`在 gb 索引中搜索 user 类型  

`/gb,us/user,tweet/_search`在 gb 和 us 索引中搜索 user 和 tweet 类型  

`/_all/user,tweet/_search`在所有的索引中搜索 user 和 tweet 类型  


参考：https://blog.csdn.net/chengyuqiang/article/details/79059958

------------------------------------------------------------

### 完整的序列说明

查看序列：

```JSON
{
    "index": {
        "aliases": {},
        "mappings": {                //映射(类型)
            "test": {              //类型type
                "properties": {
                    "date": {
                        "type": "date"
                    },
                    "message": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                    "name": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                    "user": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    }
                }
            }
        },
        "settings": {
            "index": {
                "creation_date": "1521597870261",
                "number_of_shards": "5",               #每个索引的主分片数，默认值是 5。这个配置在索引创建后不能修改
                "number_of_replicas": "1",              #每个主分片的副本数，默认值是 1。对于活动的索引库，这个配置可以随时修改
                "uuid": "grvnFKYdTyiGBk0oYu_kNw",
                "version": {
                    "created": "6020299"
                },
                "provided_name": "index"
            }
        }
    }
}
```

查询结果：

```
{
  "took": 3,                  #查询耗时
  "timed_out": false,           #查询是否超时
  "_shards": {                 #查询中参与分片的总数
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {                 #搜索结果集
    "total": 61,
    "max_score": 1,
    "hits": [
      {
        "_index": "index",
        "_type": "test",
        "_id": "79a6fc9f-fb1d-4cc7-9d18-9fe9c47e0d00",
        "_score": 1,
        "_source": {
          "user": "xiefg",
          "date": "2018-01-12",
          "message": "trying out Elasticsearch"
        }
      },
      ...
      ]
   }
}
```

>应当注意的是 timeout 不是停止执行查询，它仅仅是告知正在协调的节点返回到目前为止收集的结果并且关闭连接。
在后台，其他的分片可能仍在执行查询即使是结果已经被发送了。   
使用超时是因为 SLA(服务等级协议)对你是很重要的，而不是因为想去中止长时间运行的查询。

修改配置属性：
```
PUT /index/_settings
{
    "number_of_replicas": 1
}
```

----------------------------------------------------------------------------------------------

### 其他

#### 相关概念

节点(node)是你运行的Elasticsearch实例。一个集群(cluster)是一组具有相同cluster.name的节点集合，
他们协同工作，共享数据并提供故障转移和扩展功能，当有新的节点加入或者删除节点，集群就会感知到并平衡数据。

集群中一个节点会被选举为主节点(master),它用来管理集群中的一些变更，例如新建或删除索引、增加或移除节点等;
当然一个节点也可以组成一个集群。

我们往 Elasticsearch添加数据时需要用到索引——保存相关数据的地方。索引实际上是指向一个或者多个物理分片的逻辑命名空间。
一个分片是一个底层的工作单元，它仅保存了全部数据中的一部分。Elasticsearch 是利用分片将数据分发到集群内各处的。
分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。 

我们能够与集群中的任何节点通信，包括主节点。任何一个节点互相知道文档存在于哪个节点上，它们可以转发请求到我们需要数据所在的节点上。
我们通信的节点负责收集各节点返回的数据，最后一起返回给客户端。
这一切都由Elasticsearch透明的管理


#### 关于查询

在初始查询阶段时，查询会广播到索引中每一个分片拷贝（主分片或者副本分片）。 
每个分片在本地执行搜索并构建一个匹配文档的优先队列 。

当一个搜索请求被发送到某个节点时，这个节点就变成了协调节点。 
这个节点的任务是广播查询请求到所有相关分片并将它们的响应整合成全局排序后的结果集合，这个结果集合会返回给客户端。

![](http://p2jr3pegk.bkt.clouddn.com/elas_0901.png)

根据官方给出的图事例,可以描述该过程为：

（1）客户端发送一个 search 请求到 Node 3 ， Node 3 会创建一个空的优先队列。
（2）Node 3 将查询请求转发到索引的每个主分片或副本分片中。每个分片在本地执行查询并添加结果到本地的有序优先队列中。
（3）每个分片返回各自优先队列中所有文档的 ID 和排序值给协调节点，也就是 Node 3，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。

#### 查看集群健康

```
GET _cluster/health
```

返回：

```
{
  "cluster_name": "elasticsearch",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 56,
  "active_shards": 56,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 55,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50.45045045045045
}
```

字段含义说明:   
- green--所有的主分片和副本分片都正常运行。  
- yellow--所有的主分片都正常运行，但不是所有的副本分片都正常运行。  
- red--有主分片没能正常运行  

>我们是单节点部署elasticsearch，而默认的分片副本数目配置为1，而相同的分片不能在一个节点上，所以就存在副本分片指定不明确的问题，
所以显示为yellow,通常做法是添加一个节点来解决问题。

作为测试用，可以删除副本分片：

```
PUT /_settings
 {  "number_of_replicas" : 0 }
```


#### cat相关命令

`GET /_cat/master?v ` 查看相关配置明细

`GET /_cat/master?help` 查看相关配置明细及其含义

`GET /_cat/nodes?h=ip,port,heapPercent,name` 强制指明所有要返回的列

`GET /_cat/aliases?v` 显示相关的别名

`GET /_cat/allocation?v` 查看有多少分片分配给每个数据节点和他们使用多少磁盘空间

`GET /_cat/count?v` 对整个集群或单个索引的文档计数的快速访问

`GET /_cat/fielddata?v` 显示了当前集群中每个数据节点上的fielddata使用了多少堆内存


### BUG修复

`1、ERROR Could not register mbeans java.security.AccessControlException: access denied ("javax.management.MBeanTrustPermission" "register")`

改变 elasticsearch 文件夹所有者到当前用户：
```
###文件夹所有者到当前用户
sudo chown -R noroot:noroot elasticsearch
###给予 config 文件夹权限
sudo -i
3chmod -R 775 config
```

`2、[WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [] uncaught exception in thread [main] org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root`


原因是elasticsearch默认是不支持用root用户来启动的。

解决方案一：

`Des.insecure.allow.root=true`
修改 /usr/local/elasticsearch-2.4.0/bin/elasticsearch，

添加

`ES_JAVA_OPTS="-Des.insecure.allow.root=true"`
或执行时添加：

`sh /usr/local/elasticsearch-2.4.0/bin/elasticsearch -d -Des.insecure.allow.root=true`
注意：正式环境用root运行可能会有安全风险，不建议用root来跑。

解决方案二：添加专门的用户
```
useradd elastic
chown -R elastic:elastic  elasticsearch-2.4.0
su elastic
sh /usr/local/elasticsearch-2.4.0/bin/elasticsearch -d
```

`3、UnsupportedOperationException: seccomp unavailable: requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled in`

只是警告，使用新的linux版本，就不会出现此类问题了。

`4、ERROR: [4] bootstrap checks failed [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]`

原因：无法创建本地文件问题,用户最大可创建文件数太小

解决方案：切换到 root 用户，编辑 limits.conf 配置文件， 添加类似如下内容：

`vim /etc/security/limits.conf`
添加如下内容:
```
1* soft nofile 65536
2* hard nofile 131072
3* soft nproc 2048
4* hard nproc 4096
```
`[2]: max number of threads [1024] for user [tzs] is too low, increase to at least [2048]`

原因：无法创建本地线程问题,用户最大可创建线程数太小

解决方案：切换到root用户，进入limits.d目录下，修改90-nproc.conf 配置文件。

`vim /etc/security/limits.d/90-nproc.conf`
找到如下内容：

`soft nproc 1024`
修改为

`* soft nproc 2048`

`[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

原因：最大虚拟内存太小

root用户执行命令：

`sysctl -w vm.max_map_count=262144`
或者修改 /etc/sysctl.conf 文件，添加 “vm.max_map_count”设置
设置后，可以使用

`$ sysctl -p`

`[4]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk`

原因：Centos6不支持SecComp，而ES5.4.1默认bootstrap.system_call_filter为true进行检测，所以导致检测失败，失败后直接导致ES不能启动。
详见 ：https://github.com/elastic/elasticsearch/issues/22899

解决方法：在elasticsearch.yml中新增配置bootstrap.system_call_filter，设为false，注意要在Memory下面:
```
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```

`5、 java.lang.IllegalArgumentException: property [elasticsearch.version] is missing for plugin [head]`

再 es 的配置文件中加：
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```






