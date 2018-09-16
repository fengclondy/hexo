---
layout: post
title: ES高级篇（2）
date: 2018-03-26 09:26:13
tags: elasticsearch
categories: elasticsearch
---

>elasticsearch 6.x新特性，不在支持String类型，由 text 和 keyword 代替。
text类型一般用于全文搜索，不用于排序，很少用于聚合。
keywod类型只能通过精确值得到，一般用于排序、聚合。

### 结构化数据的部分匹配

```json
PUT /my_index
{
    "mappings": {
        "address": {
            "properties": {
                "postcode": {
                    "type":  "keyword"
                }
            }
        }
    }
}

PUT /my_index/address/1
{ "postcode": "W1V 3DG" }

PUT /my_index/address/2
{ "postcode": "W2F 8HW" }

PUT /my_index/address/3
{ "postcode": "W1F 7HW" }

PUT /my_index/address/4
{ "postcode": "WC1N 1LZ" }

PUT /my_index/address/5
{ "postcode": "SW5 0BE" }

```

<!-- more -->

#### 前缀查询

```json
GET /my_index/address/_search
{
    "query": {
        "prefix": {
            "postcode": "W1"
        }
    }
}
```

>默认状态下， prefix 查询不做相关度评分计算，它只是将所有匹配的文档返回，并为每条结果赋予评分值 1 。它的行为更像是过滤器而不是查询。 
prefix 查询和 prefix 过滤器这两者实际的区别就是过滤器是可以被缓存的，而查询不行。

#### 通配符与正则表达式查询

>与前缀查询一样，这也是底层基于词的查询。

通配符查询：

```json
GET /my_index/address/_search
{
    "query": {
        "wildcard": {
            "postcode": "W?F*HW" 
        }
    }
}
```

正则表达式查询：

```json
GET /my_index/address/_search
{
    "query": {
        "regexp": {
            "postcode": "W[0-9].+" 
        }
    }
}
```

### 查询时输入即搜索（即时搜索功能）

```json
{
    "match_phrase_prefix" : {
        "brand" : "johnnie walker bl"
    }
}
```

>这种查询的行为与 match_phrase 查询一致，不同的是它将查询字符串的最后一个词作为前缀使用

设置slop参数：

```json
{
    "match_phrase_prefix" : {
        "brand" : {
            "query": "walker johnnie bl", 
           // "slop":  10,            #使匹配时的词序有更大的灵活性，位置不固定
            "max_expansions": 50     #限制前缀扩展的影响（按字母顺序），返回50条数据
        }
    }
}
```

### Ngrams在复合词中的应用

案例：  
有些人希望在搜索 “Wörterbuch”（字典）的时候，能在结果中看到 “Aussprachewörtebuch”（发音字典）。
同样，搜索 “Adler”（鹰）的时候，能将 “Weißkopfseeadler”（秃鹰）包括在结果中。
>使用 n-gram 进行处理，然后搜索任何匹配的片段——能匹配的片段越多，文档的相关度越大.
>假设某个 n-gram 是一个词上的滑动窗口，那么任何长度的 n-gram 都可以遍历这个词。
我们既希望选择足够长的值让拆分的词项具有意义，又不至于因为太长而生成过多的唯一词

```json
PUT /my_index
{
    "settings": {
        "analysis": {
            "filter": {
                "trigrams_filter": {
                    "type":     "ngram",
                    "min_gram": 3,
                    "max_gram": 3
                }
            },
            "analyzer": {
                "trigrams": {
                    "type":      "custom",
                    "tokenizer": "standard",
                    "filter":   [
                        "lowercase",
                        "trigrams_filter"
                    ]
                }
            }
        }
    },
    "mappings": {
        "my_type": {
            "properties": {
                "text": {
                    "type":     "text",
                    "analyzer": "trigrams" 
                }
            }
        }
    }
}



POST /my_index/my_type/_bulk
{ "index": { "_id": 1 }}
{ "text": "Aussprachewörterbuch" }
{ "index": { "_id": 2 }}
{ "text": "Militärgeschichte" }
{ "index": { "_id": 3 }}
{ "text": "Weißkopfseeadler" }
{ "index": { "_id": 4 }}
{ "text": "Weltgesundheitsorganisation" }
{ "index": { "_id": 5 }}
{ "text": "Rindfleischetikettierungsüberwachungsaufgabenübertragungsgesetz" }


GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "text": "Adler"
        }
    }
}
```

### 权重的提升

1. 查询时权重的提升，使用`boost`参数，之前已经讲过。

2. 索引权重的提升,使用`indices_boost`参数：

```json
GET /docs_2014_*/_search 
{
  "indices_boost": { 
    "docs_2014_10": 3,
    "docs_2014_09": 2
  },
  "query": {
    "match": {
      "text": "quick brown fox"
    }
  }
}
```

3. 权重提升查询(boosting 查询)

```json
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "text": "apple"
        }
      },
      "negative": {
        "match": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

>只有那些匹配 positive 查询的文档罗列出来，
对于那些同时还匹配 negative 查询的文档将通过文档的原始 score 与 negative_boost相乘的方式降级后的结果.

4. constant_score 查询

>只关注有价值的查询，忽略不需要的。这里只关注wifi、garden、pool，其中最关心pool

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "description": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "garden" }}
        }},
        { "constant_score": {
          "boost":   2 
          "query": { "match": { "description": "pool" }}
        }}
      ]
    }
  }
}
```

5. 接受欢迎度提升权重

```json
PUT /blogposts/post/1
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   6
}
```

在搜索时，可以将 function_score 查询与 field_value_factor 结合使用， 即将点赞数与全文相关度评分结合：

```json
GET /blogposts/post/_search
{
  "query": {
    "function_score": {               
      "query": {                  #主查询
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {         #函数查询
        "field": "votes" 
      }
    }
  }
}
```

### 同义词

使用自定义的词汇单元过滤器，并使用同义词:

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",             #同义词类型
          "synonyms": [                #同义词格式
            "british,english",
            "queen,monarch"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter" 
          ]
        }
      }
    }
  }
}



POST /my_index/_analyze
{
  "analyzer": "my_synonyms", 
  "text": "Elizabeth is the English queen"
}


返回结果：

Pos 1: (elizabeth)
Pos 2: (is)
Pos 3: (the)
Pos 4: (british,english) 
Pos 5: (queen,monarch) 
```

### 模糊查询

```json
GET /my_index/my_type/_search
{
  "query": {
    "fuzzy": {
      "text": "surprize"
    }
  }
}


GET /my_index/my_type/_search
{
  "query": {
    "fuzzy": {
      "text": {
        "value": "surprize",
        "fuzziness": 1
      }
    }
  }
}
```

### 语音匹配

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "dbl_metaphone": { 
          "type":    "phonetic",
          "encoder": "double_metaphone"
        }
      },
      "analyzer": {
        "dbl_metaphone": {
          "tokenizer": "standard",
          "filter":    "dbl_metaphone" 
        }
      }
    }
  }
}
```

>过滤检索（Filtered query）5.0版本已不再存在，不必关注。
多个过滤器检索（Multiple Filters）5.x不再支持，无需关注。

### best fields、dis_max、

>best fields策略，就是说，搜索到的结果，应该是某一个field中匹配到了尽可能多的关键词，被排在前面；而不是尽可能多的field匹配到了少数的关键词，排在了前面
>best-fields策略，主要是说将某一个field匹配尽可能多的关键词的doc优先返回回来.(默认策略)


dis_max语法，直接取多个query中，分数最高的那一个query的分数即可

1、dis_max，只是取分数最高的那个query的分数而已。

2、dis_max只取某一个query最大的分数，完全不考虑其他query的分数

3、使用`tie_breaker`将其他query的分数也考虑进去

tie_breaker参数的意义，在于说，将其他query的分数，乘以tie_breaker，然后综合与最高分数的那个query的分数，综合在一起进行计算
除了取最高分以外，还会考虑其他的query的分数。tie_breaker的值，在0~1之间，是个小数

```json
GET /forum/article/_search
{
  "query": {
    "multi_match": {
        "query":                "java solution",
        "type":                 "best_fields",      #默认方式
        "fields":               [ "title^2", "content" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "50%" 
    }
  } 
}
```

### most_fields

most-fields策略，主要是说尽可能返回更多field匹配到某个关键词的doc，优先返回回来

```json
GET /forum/article/_search
{
   "query": {
        "multi_match": {
            "query":  "learning courses",
            "type":   "most_fields", 
            "fields": [ "sub_title", "sub_title.std" ]
        }
    }
}
```

### cross_fields

```json
  "query": {
    "multi_match": {
      "query": "Peter Smith",
      "type": "cross_fields", 
      "operator": "and",
      "fields": ["author_first_name", "author_last_name"]
    }
  }
}
```


### 用copy_to，将多个field组合成一个field，就可以将多个字段的值拷贝到一个字段中，并建立倒排索引

```json
PUT /forum/_mapping/article
{
  "properties": {
      "new_author_first_name": {
          "type":     "string",
          "copy_to":  "new_author_full_name" 
      },
      "new_author_last_name": {
          "type":     "string",
          "copy_to":  "new_author_full_name" 
      },
      "new_author_full_name": {
          "type":     "string"
      }
  }
}

POST /forum/article/_bulk
{ "update": { "_id": "1"} }
{ "doc" : {"new_author_first_name" : "Peter", "new_author_last_name" : "Smith"} }		--> Peter Smith
{ "update": { "_id": "2"} }	
{ "doc" : {"new_author_first_name" : "Smith", "new_author_last_name" : "Williams"} }		--> Smith Williams
{ "update": { "_id": "3"} }
{ "doc" : {"new_author_first_name" : "Jack", "new_author_last_name" : "Ma"} }			--> Jack Ma
{ "update": { "_id": "4"} }
{ "doc" : {"new_author_first_name" : "Robbin", "new_author_last_name" : "Li"} }			--> Robbin Li
{ "update": { "_id": "5"} }
{ "doc" : {"new_author_first_name" : "Tonny", "new_author_last_name" : "Peter Smith"} }		--> Tonny Peter Smith

GET /forum/article/_search
{
  "query": {
    "match": {
      "new_author_full_name":       "Peter Smith"
    }
  }
}
```
