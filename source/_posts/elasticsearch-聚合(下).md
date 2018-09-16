---
layout: post
title: ES高级篇 --- 聚合（下）
date: 2018-03-27 05:46:21
tags: elasticsearch
categories: elasticsearch
---


### 一个简单的聚合

>默认情况下，聚合与查询是对同一范围进行操作的，也就是说，聚合是基于我们查询匹配的文档集合进行计算的。

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color"
            }
        }
    }
}
```

>这里的`size=0`，可理解为 "没有指定查询" ，但其实Elasticsearch内部"没有指定查询"和`size!=0`("查询所有文档")是等价的。

<!--  more  -->

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "query" : {
        "match_all" : {}
    },
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color"
            }
        }
    }
}
```

>因为聚合总是对查询范围内的结果进行操作的，所以一个隔离的聚合实际上是在对 match_all 的结果范围操作，即所有的文档。

>优势：由于聚合的支持以及对查询范围的限定,用户可以搜索数据，查看所有实时更新的图形,这是 Hadoop 无法做到的。


#### 全局桶

案例：比较福特汽车与所有汽车平均售价的关系。

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "query" : {
        "match" : {
            "make" : "ford"
        }
    },
    "aggs" : {
        "single_avg_price": {              
            "avg" : { "field" : "price" }           #度量计算是基于查询范围内所有文档
        },
        "all": {
            "global" : {},                  #全局桶，没有参数
            "aggs" : {
                "avg_price": {              
                    "avg" : { "field" : "price" }         #聚合操作针对所有文档（由于全局桶的存在）
                }

            }
        }
    }
}
```

#### 过滤桶

案例1：计算售价在 $10,000 美元之上的所有汽车，并且计算这些车的平均售价

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "query" : {
        "constant_score": {
            "filter": {               #过滤桶
                "range": {
                    "price": {
                        "gte": 10000
                    }
                }
            }
        }
    },
    "aggs" : {
        "single_avg_price": {
            "avg" : { "field" : "price" }
        }
    }
}
```

案例2：创建一个搜索页面， 希望在用户进行搜索的时候，在页面上提供更丰富的信息，包括（与搜索匹配的）上个月度汽车的平均售价。

```json
GET /cars/transactions/_search
{
   "size" : 0,
   "query":{
      "match": {
         "make": "ford"
      }
   },
   "aggs":{
      "recent_sales": {
         "filter": {              #过滤桶
            "range": {
               "sold": {
                  "from": "now-1M"
               }
            }
         },
         "aggs": {
            "average_price":{
               "avg": {
                  "field": "price" 
               }
            }
         }
      }
   }
}
```

>filter 桶和其他桶的操作方式一样，可以随意将其他桶和度量嵌入其中。
所有嵌套的组件都会 "继承" 这个过滤，所以可以按需针对聚合过滤出选择部分。


#### 后过滤器

>`post_filter`这个过滤器在查询之后执行的，所以不会对查询范围和聚合结果有任何影响，一般与聚合一起使用。

案例1：展示所有的绿色 ford 汽车

```json
GET /cars/transactions/_search
{
    "query": {
        "match": {
            "make": "ford"
        }
    },
    "aggs" : {
        "all_colors": {
            "terms" : { "field" : "color.keyword" }
        }
    },
    "post_filter": {         #后过滤
        "term" : {
            "color" : "green"
        }
    }
}
```

>分析：1）查询所有的文档，找到ford汽车的文档。2）用terms聚合创建一个关于ford汽车的颜色列表，即颜色列表和汽车颜色对应。
3）使用后过滤，在查询结果里过滤掉非绿色的汽车。

>可能没太懂。一句话总结就是：对搜索结果和聚合结果进行过滤（使用filter查询），
对聚合结果的一部分进行过滤（使用filter桶），只过滤搜索结果，不过滤聚合结果（使用post_filter）


### 扩展属性

#### 内置排序

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color.keyword",
              "order": {                 #排序参数,不可变
                "_count" : "asc"          #默认的降序，此处改为了升序
              }
            }
        }
    }
}
```

>`_count`按文档数排序。对 terms 、 histogram 、 date_histogram 有效。  
`_term`按词项的字符串值的字母顺序排序。只在 terms 内使用。   
`_key`按每个桶的键值数值排序（理论上与` _term `类似）。 只在 histogram 和 date_histogram 内使用。

当然，我们也可以自定义排序规则：

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color",
              "order": {
                "avg_price" : "asc"           #按照计算平均值的升序排序
              }
            },
            "aggs": {
                "avg_price": {
                    "avg": {"field": "price"} 
                }
            }
        }
    }
}
```

如果含有多个度量值，需以关心的度量为关键词使用点式路径。

案例1：按每个桶的方差来进行升序排序

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color.keyword",
              "order": {
                "stats.variance" : "asc" 
              }
            },
            "aggs": {
                "stats": {
                    "extended_stats": {"field": "price"}
                }
            }
        }
    }
}
```

#### 基于深度度量排序

>目前，只有三个单值桶： filter 、 global 和 reverse_nested, 可以定义更深的路径，
将度量用尖括号（ > ）嵌套起来，然后使用。

案例1：创建一个汽车售价的直方图，按照红色和绿色（不包括蓝色）车各自的方差来进行排序。

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "histogram" : {
              "field" : "price",
              "interval": 20000,
              "order": {
                "red_green_cars>stats.variance" : "asc"       #按照嵌套度量的方差对桶的直方图进行排序
              }
            },
            "aggs": {
                "red_green_cars": {
                    "filter": { "terms": {"color": ["red", "green"]}},     #使用了单值过滤器 filter ，所以可用嵌套排序
                    "aggs": {
                        "stats": {"extended_stats": {"field" : "price"}} 
                    }
                }
            }
        }
    }
}
```

>分析：stats 度量是 red_green_cars 聚合的子节点，而 red_green_cars 又是 colors 聚合的子节点。 
为了根据这个度量排序，我们定义了路径 red_green_cars>stats.variance 。
我们可以这么做，因为 filter 桶是个单值桶。


#### 算法

>Elasticsearch 目前支持两种近似算法（ cardinality 和 percentiles ）

案例1：计算出每月有多少颜色的车被售出。

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "distinct_colors" : {
            "cardinality" : {                   #基数度量
              "field" : "color.keyword",
              "precision_threshold" : 100         #设定阈值大小，可不设定，有默认值
            }
        }
    }
}
```

返回结果：

```json
"aggregations": {
    "months": {
      "buckets": [
        {
          "key_as_string": "2014-01-01T00:00:00.000Z",
          "key": 1388534400000,
          "doc_count": 1,
          "distinct_colors": {
            "value": 1
          }
        },
        ...
        
```

##### 速度优化
```

```