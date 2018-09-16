---
layout: post
title: ES之全文所有引擎的搭建
date: 2018-03-23 10:15:51
tags: elasticsearch
categories: elasticsearch
---

参考：http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html

>ik分词：
`ik_max_word `：会将文本做最细粒度的拆分；尽可能多的拆分出词语
`ik_smart`：会做最粗粒度的拆分；已被分出的词语将不会再次被其它词语占有

### 创建

>新建一个名称为`accounts`的 Index，里面有一个名称为`person`的 Type。person有三个字段`user`、`title`、`desc`
指定中文分词器，不使用默认的英文分词器

设置:

```json
PUT accounts
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",    //字段文本的分词器
          "search_analyzer": "ik_max_word"  //搜索词的分词器（对文本进行最大数量的分词）
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}
```

<!-- more -->

插入数据：

```json
POST /_bulk
{ "create":{ "_index": "accounts", "_type": "person", "_id": "4" }}
{ "user":"张三","title":"CentOS7.x下的Ambari2.4源码编译","desc":"http://url.cn/53788351"}
{ "create":{ "_index": "accounts", "_type": "person", "_id": "5" }}
{ "user":"李四","title":"CentOS7.x的watchman源码","desc":"https://url.cn/53844169"}
{ "create":{ "_index": "accounts", "_type": "person", "_id": "6" }}
{ "user":"李四","title":"CentOS7.x的watchman编译","desc":"https://url.cn/53844169"}
```

查询匹配结果：

```json
GET accounts/_search
{
  "query": {
    "match": {
      "title": {
        "query": "源码编译"
      }
    }
  }
}
```


### 批量导入本地数据

1、下载官方网站的三个数据源文件并解压accounts.json、shakespeare_6.0.json、logs.jsonl

2、创建映射关系：
```json
## 该文件的前两个字段不分析，后两字段是数值型数据
PUT /shakespeare
{
 "mappings": {
  "doc": {
   "properties": {
    "speaker": {"type": "keyword"},
    "play_name": {"type": "keyword"},
    "line_id": {"type": "integer"},
    "speech_number": {"type": "integer"}
   }
  }
 }
}
```

```json
## 日志文件的 coordinates字段记录地理位置信息
PUT /logstash-2015.05.18
{
  "mappings": {
    "log": {
      "properties": {
        "geo": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  }
}

PUT /logstash-2015.05.19
{
  "mappings": {
    "log": {
      "properties": {
        "geo": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  }
}

PUT /logstash-2015.05.20
{
  "mappings": {
    "log": {
      "properties": {
        "geo": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  }
}
```

3、在该文件夹下导入数据：
```shell
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/_bulk?pretty' --data-binary @logs.jsonl
```
>如果是windows环境下，一定要将上述的‘’转换成“”，否则会报`curl: (6) Could not resolve host: application`这样的错误。




### 全文检索
https://www.elastic.co/guide/en/elasticsearch/reference/6.1/full-text-queries.html

#### match匹配查询

- 普通查询
```json
GET website/_search
{
  "query": {
    "match": {
        "title":{
          "query":"centos升级"
        }
    }
  }
}
```
或者：
```json
GET website/_search
{
  "query": {
    "match": {
        "title": "centos升级"
    }
  }
}
```
- and操作

```json
GET website/_search
{
  "query": {
    "match": {
        "title":{
          "query":"centos升级",
          "operator":"and"
        }
    }
  }
}
```

- or操作

```json
GET website/_search
{
  "query": {
    "match": {
        "title":{
          "query":"centos升级",
          "operator":"or"
        }
    }
  }
}
```

#### match_phrase查询（短语查询）

>不区分大小写。

>与match query类似，但用于匹配精确短语，可称为短语查询。match_phrase查询会将查询内容分词，分词器可以自定义，文档中同时满足以下两个条件才会被检索到：
（1）分词后所有词项都要出现在该字段中；
（2）字段中的词项顺序要一致.
```json
GET test/_search
{
  "query": {
    "match_phrase": {
      "content": "hello world"
    }
  }
}
```

#### match_phrase_prefix 查询（前缀查询）
>不区分大小写。

>match_phrase_prefix与match_phrase相同，只是它允许在文本中的最后一个词的前缀匹配。
也就是说，对match_phrase进行了扩展，查询内容的最后一个分词与只要满足前缀匹配即可。

```json
GET test/_search
{
  "query": {
    "match_phrase_prefix": {
      "content": "hello wor"
    }
  }
}
```
#### multi_match 查询
multi_match查询是match查询的升级版，用于多字段检索。只要有一个匹配就被查询出来。

```json
GET website/_search
{
  "query": {
    "multi_match": {
      "query": "centos",
      "fields": ["title","abstract"]
    }
  }
}
```
#### common_terms 查询（常用词查询）
（1）停用词。
有些词在文本中出现的频率非常高，但是对文本所携带的信息基本不产生影响。
比如英文中的a、an、the、of，中文的“的”、”了”、”着”、”是” 、标点符号等。
文本经过分词之后，停用词通常被过滤掉，不会被进行索引。在检索的时候，用户的查询中如果含有停用词，检索系统也会将其过滤掉（因为用户输入的查询字符串也要进行分词处理）。
排除停用词可以加快建立索引的速度，减小索引库文件的大小。

（2）虽然停用词对文档评分影响不大，但是有时停用词仍然具有重要意义，去除停用词显然不合适。
如果去除停用词，就无法区分“happy”和”not happy”, “to be or not to be”就不能被索引，搜索的准确率就会降低。

（3）common_terms查询提供了一种解决方案，把查询分词后的词项分为重要词项(比如low frequency terms ,低频词)和不重要词(high frequency terms which would previously have been stopwords,高频的停用词)。
在搜索时，首先搜索与重要词匹配的文档，然后执行第二次搜索，搜索评分较小的高频词。
Terms are allocated to the high or low frequency groups based on the cutoff_frequency, which can be specified as an absolute frequency (>=1) or as a relative frequency (0.0 .. 1.0).
词项是高频词还是低频词，可以通过cutoff_frequency来设置阀值，取值可以是绝对频率 (>=1)或者相对频率(0.0 ~1.0)
```json
GET website/_search
{
    "query": {
        "common": {
            "title": {
                "query": "to be",
                "cutoff_frequency": 0.0001,                 #设置阈值
                "low_freq_operator": "and"                   #设置低频词
            }
        }
    }
}
```

#### query_string查询
query_string查询与Lucence查询语句紧密结合，允许在一个查询语句中使用多个特殊条件关键字，建议熟悉Lucence查询语法用户使用。

#### simple_query_string
解析出错时不抛异常，丢弃查询无效的部分

```json
GET website/_search
{
  "query": {
    "simple_query_string" : {
        "query": "\"fried eggs\" +(eggplant | potato) -frittata",
        "fields": ["title^5", "abstract"],
        "default_operator": "and"
    }
  }
}
```

#### 词项查询
全文查询将在执行之前分析查询字符串，但词项级别查询将按照存储在倒排索引中的词项进行精确操作。

https://www.elastic.co/guide/en/elasticsearch/reference/6.1/term-level-queries.html

#### term查询
term 查询被用于精确值匹配，这些精确值可能是数字、时间、布尔型或者keyword，不对文本进行分析。

#### terms查询
terms 查询和 term 查询一样，但它允许你指定多值进行匹配。

#### terms_set查询
terms_set查询是一个新的查询，它的语法将来可能会改变。查找与一个或多个指定词项匹配的文档，其中必须匹配的术语数量取决于指定的最小值，应匹配字段或脚本。

```json
PUT my-index
{
    "mappings": {
        "doc": {
            "properties": {
                "required_matches": {
                    "type": "long"
                }
            }
        }
    }
}
```
```json
PUT /my-index/doc/1?refresh
{
    "codes": ["ghi", "jkl"],
    "required_matches": 2
}

PUT /my-index/doc/2?refresh
{
    "codes": ["def", "ghi"],
    "required_matches": 2
}
```
最小值匹配的查询:
```json
GET /my-index/_search
{
    "query": {
        "terms_set": {
            "codes" : {
                "terms" : ["abc", "def", "ghi"],                          #匹配的值
                "minimum_should_match_field": "required_matches"         #匹配字段
            }
        }
    }
}
```
返回：
```json
{
  "took": 66,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "my-index",
        "_type": "doc",
        "_id": "2",
        "_score": 0.5753642,
        "_source": {
          "codes": [
            "def",
            "ghi"
          ],
          "required_matches": 2
        }
      }
    ]
  }
}
```
一个总是限制匹配条件数量永远不会超过指定词项数量的例子如下，其中params.num_terms参数在脚本中可用，以指示已指定的词项数。

```json
GET /my-index/_search
{
    "query": {
        "terms_set": {
            "codes" : {
                "terms" : ["abc", "def", "ghi"],
                "minimum_should_match_script": {
                   "source": "Math.min(params.num_terms, doc['required_matches'].value)"
                }
            }
        }
    }
}
```
返回：

```json
{
  "took": 282,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "my-index",
        "_type": "doc",
        "_id": "2",
        "_score": 0.5753642,
        "_source": {
          "codes": [
            "def",
            "ghi"
          ],
          "required_matches": 2
        }
      }
    ]
  }
}
```

#### range查询
range查询用于匹配数值型、日期型或字符串型字段在某一范围内的文档。

>`gt`大于;
`gte`大于等于;
`lt`小于;
`lte`小于等于.

例子1：搜索age字段在10到20的所有文档:
```json
DELETE my-index

PUT my-index

PUT my-index/doc/1
{"age":12}

PUT my-index/doc/2
{"age":18}

PUT my-index/doc/3
{"age":21}

```
```json
GET _search
{
    "query": {
        "range" : {
            "age" : {
                "gte" : 10,
                "lte" : 20,
                "boost" : 2.0
            }
        }
    }
}
```
返回：
```json
{
  "took": 24,
  "timed_out": false,
  "_shards": {
    "total": 55,
    "successful": 55,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 2,
    "hits": [
      {
        "_index": "my-index",
        "_type": "doc",
        "_id": "2",
        "_score": 2,
        "_source": {
          "age": 18
        }
      },
      {
        "_index": "my-index",
        "_type": "doc",
        "_id": "1",
        "_score": 2,
        "_source": {
          "age": 12
        }
      }
    ]
  }
}
```
例子2：日期范围查询:

```json
GET website/_search
{
    "query": {
        "range" : {
            "postdate" : {
                "gte" : "2017-01-01",
                "lte" :  "2017-12-31",
                "format": "yyyy-MM-dd"
            }
        }
    }
}
```
返回：

```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
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
      }
    ]
  }
}
```

#### exists查询和missing查询
返回原始字段中至少包含一个非空值的文档。

```json
PUT my-index/doc/1
{ "user": "jane" }

PUT my-index/doc/2
{ "user": "" } 

PUT my-index/doc/3
{ "user": [] }

PUT my-index/doc/4
{ "user": ["jane", null ] }

PUT my-index/doc/5
{ "age": 28 }

GET /_search
{
    "query": {
        "exists" : { "field" : "user" }
    }
}
```
返回：
```json
{
  "took": 18,
  "timed_out": false,
  "_shards": {
    "total": 55,
    "successful": 55,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "my-index",
        "_type": "doc",
        "_id": "2",
        "_score": 1,
        "_source": {
          "user": ""
        }
      },
      {
        "_index": "my-index",
        "_type": "doc",
        "_id": "4",
        "_score": 1,
        "_source": {
          "user": [
            "jane",
            null
          ]
        }
      },
      {
        "_index": "my-index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "user": "jane"
        }
      }
    ]
  }
}
```

>说明：
可匹配的查询：
1.”user”: “” ，有user字段，值非空（空字符串）
2.”user”: “jane”，有user字段，值非空
3.”user”: [“jane”,null]，有user字段，至少有一个值非空
不能匹配的文档：
1.”user”: []，有user字段，值空
2.”age”: 28，没有user字段

#### prefix查询
例子1：查询以ki开头的用户
```json
GET /_search
{ "query": {
    "prefix" : { "user" : "ki" }
  }
}
```
#### wildcard查询（通配符查询）

```json
GET website/_search
{
    "query": {
        "wildcard" : { "title" : "*yum*" }
    }
}
```

#### regexp查询（正则表达式查询）

```json
GET website/_search
{
    "query": {
        "regexp":{
            "title": "gc.*"
        }
    }
}
```
#### fuzzy查询（模糊查询）
```json
GET website/_search
{
    "query": {
        "fuzzy":{
            "title": "vmwere"
        }
    }
}
```
#### type查询
```json
GET /_search
{
    "query": {
        "type" : {
            "value" : "my_type"
        }
    }
}
```
#### ids查询
```json
GET /_search
{
    "query": {
        "ids" : {
            "type" : "blog",
            "values" : ["2", "3"]
        }
    }
}
```
https://blog.csdn.net/chengyuqiang/article/details/79117081




