---
layout: post
title: elasticsearch索引分词 ---简单的操作案例
date: 2018-03-22 10:47:15
tags: elasticsearch
categories: elasticsearch
---

>从ElasticSearch 5.x开始不再支持string，由text和keyword类型替代。  
当一个字段是要被全文搜索的，比如Email内容、产品描述，应该使用text类型。  
设置text类型以后，字段内容会被分析，在生成倒排索引以前，字符串会被分析器分成一个一个词项。  
text类型的字段不用于排序，很少用于聚合。  
keyword类型适用于索引结构化的字段，比如email地址、主机名、状态码和标签。  
如果字段需要进行过滤(比如查找已发布博客中status属性为published的文章)、排序、聚合。  
keyword类型的字段只能通过精确值搜索到。  

>另外一个重要的改变是：6.x后一个索引只能对应一种类型，不存在一个索引有多重类型的情况。
官方给出的解释是：最初，SQL数据库中的“数据库”类似的“索引”，“类型”与“表”相当的假设是错误的，因为两个表中的同名字段并没有关系，属于设计缺陷，现在统一去除了。

>>所以序列和类型的组合是唯一对应的（单向唯一性约束）。 即，创建时必须指明序列-类型，查询/删除时可以直接写序列，而不写类型，索引不能为空

<!-- more -->

#### 1、创建索引 (可以采用默认的，跳过该步骤)

```json
PUT website
{
  "settings": {
    "number_of_replicas": 1,
    "number_of_shards": 5
  },
  "mappings": {
    "blog":{
      "properties": {
        "title":{
          "type":"text",
          "analyzer": "ik_max_word",
          //"fields": {                          #最新6.X版如果不自己配置，默认会加上子类型，这里推荐都加上
              //"keyword": {
               // "type": "keyword",
               // "ignore_above": 256
             // }
          //  }
        },
        "author":{
          "type":"text"
        },
        "postdate":{
          "type":"date",
          "format": "yyyy-MM-dd"
        },
        "abstract":{
          "type":"text",
          "analyzer": "ik_max_word"
        },
        "url":{
          "type":"text"
        }
      }
    }  
  }
}
```

#### 2、批量导入数据

```json
POST /_bulk
{ "create":{ "_index": "website", "_type": "blog", "_id": "1" }}
{ "title": "Ambari源码编译","author":"程裕强","postdate":"2016-12-21","abstract":"CentOS7.x下的Ambari2.4源码编译","url":"http://url.cn/53788351"}
{ "create":{ "_index": "website", "_type": "blog", "_id": "2" }} 
{ "title": "watchman源码编译","author":"程裕强","postdate":"2016-12-23","abstract":"CentOS7.x的watchman源码编译","url":"http://url.cn/53844169"}
{ "create":{ "_index": "website", "_type": "blog", "_id": "3" }}
{ "title": "CentOS升级gcc","author":"程裕强","postdate":"2016-12-25","abstract":"CentOS升级gcc","url":"http://url.cn/53868915"}
{ "create":{ "_index": "website", "_type": "blog", "_id": "4" }}
{ "title": "vmware复制虚拟机","author":"程裕强","postdate":"2016-12-29","abstract":"vmware复制虚拟机","url":"http://url.cn/53946664"}
{ "create":{ "_index": "website", "_type": "blog", "_id": "5" }}
{ "title": "libstdc++.so.6","author":"程裕强","postdate":"2016-12-30","abstract":"libstdc++.so.6问题解决","url":"http://url.cn/53946911"}
{ "create":{ "_index": "website", "_type": "blog", "_id": "6" }}
{ "title": "CentOS更换国内yum源","author":"程裕强","postdate":"2016-12-30","abstract":"CentOS更换国内yum源","url":"http://url.cn/53946911"}
{ "create":{ "_index": "website", "_type": "blog", "_id": "7" }}
{ "title": "搭建Ember开发环境","author":"程裕强","postdate":"2016-12-30","abstract":"CentOS下搭建Ember开发环境","url":"http://url.cn/53947507"}
{ "create":{ "_index": "website", "_type": "blog", "_id": "8" }}
{ "title": "es高亮","author":"程裕强","postdate":"2017-01-03","abstract":"Elasticsearch查询关键字高亮","url":"http://url/53991802"}
{ "create":{ "_index": "website", "_type": "blog", "_id": "9" }}
{ "title": "to be or not to be","author":"somebody","postdate":"2018-01-03","abstract":"to be or not to be,that is the question","url":"http://url/63991802"}
```

返回：

```json
{
  "took" : 28,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "1",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "2",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "3",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "4",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "5",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "6",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "7",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "8",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "9",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```

#### 3、查询全部数据：

```json
GET website/_search?q=*
```

或者使用：

```json
GET website/_search
{
    "query": {
        "match_all": {}
    }
}
```
或者：
```json
GET website/_search
```

#### 4、term 查询(精确值查询)

```json
GET website/_search
{
  "query": {
    "term": {
        "title": "vmware"
    }
  }
}
```

返回：

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.9227539,
    "hits": [
      {
        "_index": "website",
        "_type": "blog",
        "_id": "4",
        "_score": 0.9227539,
        "_source": {
          "title": "vmware复制虚拟机",
          "author": "程裕强",
          "postdate": "2016-12-29",
          "abstract": "vmware复制虚拟机",
          "url": "http://url.cn/53946664"
        }
      }
    ]
  }
}
```

如果采用默认的配置，text类型下会细分keyword，则查询：
```json
# 通常当查找一个精确值的时候，我们不希望对查询进行评分计算。只希望对文档进行包括或排除的计算，
# 所以我们会使用 constant_score 查询以非评分模式来执行 term 查询并以1作为统一评分。
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "title.keyword" : "vmware复制虚拟机"
                }
            }
        }
    }
}

# 设置不对某一个字段进行分析处理，从而精确查找（否则默认会进行拆分处理，形如：XHDK-A-1293-#fJ3）

# 设置不被检索字段
PUT /my_store
{
    "mappings" : {
        "products" : {
            "properties" : {
                "productID" : {
                    "type" : "text",
                    "index" : "false"     #指明该字段不能被检索到
                }
            }
        }
    }
}
```

#### 5、分页

```json
GET website/_search
{
  "from":0,
  "size":3,
  "query": {
    "match_all": {}
  }
}
```

返回：

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 9,
    "max_score": 1,
    "hits": [
      {
        "_index": "website",
        "_type": "blog",
        "_id": "5",
        "_score": 1,
        "_source": {
          "title": "libstdc++.so.6",
          "author": "程裕强",
          "postdate": "2016-12-30",
          "abstract": "libstdc++.so.6问题解决",
          "url": "http://url.cn/53946911"
        }
      },
      {
        "_index": "website",
        "_type": "blog",
        "_id": "8",
        "_score": 1,
        "_source": {
          "title": "es高亮",
          "author": "程裕强",
          "postdate": "2017-01-03",
          "abstract": "Elasticsearch查询关键字高亮",
          "url": "http://url/53991802"
        }
      },
      {
        "_index": "website",
        "_type": "blog",
        "_id": "9",
        "_score": 1,
        "_source": {
          "title": "to be or not to be",
          "author": "somebody",
          "postdate": "2018-01-03",
          "abstract": "to be or not to be,that is the question",
          "url": "http://url/63991802"
        }
      }
    ]
  }
}
```

#### 6、过滤字段

```json
GET website/_search
{
  "_source": ["title","author"], 
  "query": {
    "term": {
        "title": "centos"
    }
  }
}
```

返回：

```json
{
  "took": 55,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.9227539,
    "hits": [
      {
        "_index": "website",
        "_type": "blog",
        "_id": "6",
        "_score": 0.9227539,
        "_source": {
          "author": "程裕强",
          "title": "CentOS更换国内yum源"
        }
      },
      {
        "_index": "website",
        "_type": "blog",
        "_id": "3",
        "_score": 0.2876821,
        "_source": {
          "author": "程裕强",
          "title": "CentOS升级gcc"
        }
      }
    ]
  }
}
```

#### 7、显示version

```json
GET website/_search
{
  "_source": ["title"], 
  "version": true, 
  "query": {
    "term": {
        "title": "centos"
    }
  }
}
```

#### 8、评分过滤

```json
GET website/_search
{
  "min_score":"0.5",
  "query": {
    "term": {
        "title": "centos"
    }
  }
}
```

返回：

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.9227539,
    "hits": [
      {
        "_index": "website",
        "_type": "blog",
        "_id": "6",
        "_score": 0.9227539,
        "_source": {
          "title": "CentOS更换国内yum源",
          "author": "程裕强",
          "postdate": "2016-12-30",
          "abstract": "CentOS更换国内yum源",
          "url": "http://url.cn/53946911"
        }
      }
    ]
  }
}
```

#### 9、高亮关键字

```json
GET website/_search
{
  "query": {
    "term": {
        "title": "centos"
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    }
  }
}
```
>这里的term能够查询出来是因为有分词的存在（中英文是分开的）,centos作为一个完整的个体

返回：

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.9227539,
    "hits": [
      {
        "_index": "website",
        "_type": "blog",
        "_id": "6",
        "_score": 0.9227539,
        "_source": {
          "title": "CentOS更换国内yum源",
          "author": "程裕强",
          "postdate": "2016-12-30",
          "abstract": "CentOS更换国内yum源",
          "url": "http://url.cn/53946911"
        },
        "highlight": {
          "title": [
            "<em>CentOS</em>更换国内yum源"
          ]
        }
      },
      {
        "_index": "website",
        "_type": "blog",
        "_id": "3",
        "_score": 0.2876821,
        "_source": {
          "title": "CentOS升级gcc",
          "author": "程裕强",
          "postdate": "2016-12-25",
          "abstract": "CentOS升级gcc",
          "url": "http://url.cn/53868915"
        },
        "highlight": {
          "title": [
            "<em>CentOS</em>升级gcc"
          ]
        }
      }
    ]
  }
}
```

也可以使用自定义的方式：

```json
GET website/_search
{
  "query": {
    "match": {
      "title": "查"
    }
  },
  "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "title" : {}
        }
    }
}
```

