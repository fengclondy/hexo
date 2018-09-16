---
layout: post
title: ES高级篇(1)
date: 2018-03-26 02:18:35
tags: elasticsearch
categories: elasticsearch
---

>几点注意事项：
> - 1、高亮字段的field一定要出现在查询语句中，否则不会显示高亮字段。
> - 2、高亮显示一般只针对text类型的字段，即可以被分词的字段。其它字段并不支持。


参考[官方文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_how_match_uses_bool.html)

### 精确值查找

```json
POST /my_store/products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20, "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30, "productID" : "QQPX-R-3956-#aD8" }

# 通常当查找一个精确值的时候，我们不希望对查询进行评分计算。只希望对文档进行包括或排除的计算，
# 所以我们会使用 constant_score 查询以非评分模式来执行 term 查询并以1作为统一评分。
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "productID.keyword" : "XHDK-A-1293-#fJ3"
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

<!-- more -->

补充:批量更新：

```json
POST /my_store/products/_bulk
{ "update": { "_id": 1 }}
{ "doc" : { "price" : 10, "productID" : "XHDK-A-1293-#fJ3", "msg" : "01" }}
{ "update": { "_id": 2 }}
{ "doc" : { "price" : 20, "productID" : "KDKE-B-9947-#kL5", "msg" : "02"  }}
{ "update": { "_id": 3 }}
{ "doc" : { "price" : 30, "productID" : "JODL-X-1937-#pV7", "msg" : "03"  }}
{ "update": { "_id": 4 }}
{ "doc" : { "price" : 30, "productID" : "QQPX-R-3956-#aD8", "msg" : "04"  }}
```

### 组合过滤器

标准格式:

```json
{
   "bool" : {
      "must" :     [],
      "should" :   [],
      "must_not" : [],
   }
}
```

>`must`所有的语句都必须（must）匹配，与 AND 等价。
`must_not`所有的语句都不能（must not）匹配，与 NOT 等价。
`should`至少有一个语句要匹配，与 OR 等价。

```json
GET /my_store/products/_search
{
   "query" : {
      "bool" : {
         "should" : [
           { "term" : {"price" : 20}}, 
           { "term" : {"productID.keyword" : "XHDK-A-1293-#fJ3"}} 
         ],
         "must_not" : {
            "term" : {"price" : 30} 
         }
      }
   }
}
```

复杂:

```json
GET /my_store/products/_search
{
   "query" : {
      "bool" : {
        "should" : [
          { "term" : {"productID.keyword" : "KDKE-B-9947-#kL5"}}, 
          { "bool" : { 
            "must" : [
              { "term" : {"productID.keyword" : "JODL-X-1937-#pV7"}}, 
              { "term" : {"price" : 30}} 
            ]
          }}
        ]
      }
    }
}
```

term 和 terms 是包含(contains)操作，而非等值(equals)(判断),
会构造一个 bitset标记（用于判断还有几个匹配选项），然后将所有匹配返回。
为实现精确查找，需增加 tag_count 字段(可理解为返回的个数)

```json
GET /my_index/my_type/_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                 "bool" : {
                    "must" : [
                        { "term" : { "tags" : "search" } }, 
                        { "term" : { "tag_count" : 1 } } 
                    ]
                }
            }
        }
    }
}
```

>格式说明：  
>term: {"field": "value"}  
terms: {"field": ["value1", "value2"]}


### 范围查找

```json
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "price" : {
                        "gte" : 20,
                        "lt"  : 40
                    }
                }
            }
        }
    }
}
```

>`gt`: > 大于（greater than）  
`lt`: < 小于（less than）  
`gte`: >= 大于或等于（greater than or equal to）  
`lte`: <= 小于或等于（less than or equal to）  

对日期进行操作的时候，除了标准格式（2018-03-26 00:00:00）外，还可以：
```json
"range" : {
    "timestamp" : {
        "gt" : "now-1h"      #过去一小时的
    }
}
```
```json
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-01 00:00:00||+1M"    //加一个月
    }
}
```

>第二种方式可能运行会出错，因为默认的date数据是时间轴格式的，非标准格式。
（一般，在创建时需指明date为标准格式）
```json
...
"date": {
     "type":   "date",
     "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
 }
```
如果未指明标准格式，这里就不可以用标准格式进行判断，需用时间轴：

```json
  "lt": "1528794698000"
```


### 非空（NULL）查询
```json
POST /my_index/posts/_bulk
{ "index": { "_id": "1" } }
{ "tags" : ["search"] }  
{ "index": { "_id": "2" } }
{ "tags" : ["search", "open_source"] }
{ "index": { "_id": "3" } }
{ "other_field" : "some data" }  
{ "index": { "_id": "4" } }
{ "tags" : null }  
{ "index": { "_id": "5" } }
{ "tags" : ["search", null] }  


GET /my_index/posts/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "exists" : { "field" : "tags" }
            }
        }
    }
}
```
此处，可有另一个查询，缺失查询（即不存在该字段或者该字段为null）：
```json
GET /my_index/posts/_search
{
    "query" : {
        "constant_score" : {
            "filter": {
                "missing" : { "field" : "tags" }
            }
        }
    }
}
```
当然，上面的查询也可以对对象的内部进行判断，如`{ "exists": { "field": "name.first" }}`


### 多次查询
```json
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "BROWN DOG!"
        }
    }
}
```
match 查询必须查找两个词（ ["brown","dog"] ），内部实际上先执行两次 term 查询(这里一定要注意而非一次)，
然后将两次查询的结果合并作为最终结果输出。

match 查询还可以接受 operator 操作符作为输入参数，默认情况下该操作符是 or 。
我们可以将它修改成 and 让所有指定词项都必须匹配。
```json
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": {      
                "query":    "BROWN DOG!",
                "operator": "and"
            }
        }
    }
}
```
还可以控制查询的精度：
```json
GET /my_index/my_type/_search
{
  "query": {
    "match": {
      "title": {
        "query":                "quick brown dog",
        "minimum_should_match": "75%"
      }
    }
  }
}
```
>这里的值必须加引号，且只能用百分数形式，不能用小数。

相对复杂的例子。
content 字段必须包含 full 、 text 和 search 所有三个词。
如果 content 字段也包含 Elasticsearch 或 Lucene ，包含（Elasticsearch）文档会获得更高的评分。
```json
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {  
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3             #该字段的默认值为1，加上则具有更高的重要性
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 
                    }
                }}
            ]
        }
    }
}
```

根据语句权重和优先级的复杂语句.(处于同一层的每条语句具有相同的权重，这就是为什么bool里面包含bool语句的原因)

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { 
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { 
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { 
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
```



### 控制分析
默认采用 default 的分析器（即standard 标准分析器），也可以自定义 字段的 analyzer
```json
PUT website
{    
    "mappings": {
        "blog":{
          "properties": {
            "title":{
                  "type":"text",
                  "analyzer": "ik_max_word"
                },
             "author":{
                  "type":"text"
                },
             "postdate":{
                  "type":"date",
                  "format": "yyyy-MM-dd"
                }
            }
        }
    }
}
```

### 最大化查询（dis_max查询）
```json
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```
>同时包含 brown 和 fox 的单个字段比反复出现相同词语的多个不同字段有更高的相关度
```json
{
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}
```
>指定 tie_breaker 这个参数将其他匹配语句的评分也考虑其中。步骤：（1）获得最佳匹配语句的评分。
（2）将其他匹配语句的评分结果与 tie_breaker 相乘。（3）对以上评分求和并规范化。该参数介于0-1，一般取值0.1-0.4，应根据实际情况微调

使用multi_match，简化上面的形式：
```json
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields",   #默认方式
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" 
    }
}
```
关于multi_match,一些基本的操作： 

使用了模糊正则匹配
```json
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}
```
并提升了chapter_title的权重为2,默认的权重都为1
```json
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ]    
    }
}
```

### 近似匹配

#### 短语匹配
```json
GET /my_index/my_type/_search
{
    "query": {
        "match_phrase": {
            "title": "quick brown fox"
        }
    }
}
```
等价于：
```json
"match": {
    "title": {
        "query": "quick brown fox",
        "type":  "phrase"
    }
}
```
还有一种特殊的混合匹配查询：
```json
GET /my_index/my_type/_search
{
    "query": {
        "match_phrase": {
            "title": {
                "query": "quick fox",
                "slop":  1
            }
        }
    }
}
```
>slop 参数告诉 match_phrase 查询移动多少次能将文档视为匹配。  
quick fox要将fox向右移动一个位置才能与quick brown fox匹配 
```json
DELETE /my_index/groups/ 

PUT /my_index/_mapping/groups 
{
    "properties": {
        "names": {
            "type":                "string",
            "position_increment_gap": 100
        }
    }
}
```
>position_increment_gap 设置告诉 Elasticsearch 应该为数组中每个新元素增加当前词条 position 的指定值。

注意：一个简单的 term 查询比一个短语查询大约快 10 倍，比邻近查询(有 slop 的短语 查询)大约快 20 倍。
当然，这个代价指的是在搜索时而不是索引时

#### 结果集重新评分
```json
GET /my_index/my_type/_search
{
    "query": {
        "match": {  
            "title": {
                "query":                "quick brown fox",
                "minimum_should_match": "30%"
            }
        }
    },
    "rescore": {
        "window_size": 50,         #重新评分的顶部文档数量，当超过改值时会进行另一个查询
        "query": {         
            "rescore_query": {
                "match_phrase": {
                    "title": {
                        "query": "quick brown fox",
                        "slop":  50
                    }
                }
            }
        }
    }
}
```

#### 使用single词汇单元过滤器
```json
DELETE /my_index

PUT /my_index
{
    "settings": {
        "number_of_shards": 1,  
        
        "analysis": {
            "filter": {
                "my_shingle_filter": {
                    "type":             "shingle",    #使用词汇
                    "min_shingle_size": 2,        #最小词汇长度，默认是2
                    "max_shingle_size": 2,        #最大词汇长度，默认是2
                    "output_unigrams":  false     #想让 unigrams 和 bigrams 分开，即词汇分开
                }
            },
            "analyzer": {
                "my_shingle_analyzer": {
                    "type":             "custom",
                    "tokenizer":        "standard",
                    "filter": [
                        "lowercase",
                        "my_shingle_filter"       #使用常规语汇单元过滤器
                    ]
                }
            }
        }
    }
}
```