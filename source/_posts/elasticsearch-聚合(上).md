---
layout: post
title: elasticsearchh高级篇 --- 聚合(上)
date: 2018-03-27 01:59:36
tags: elasticsearch
categories: elasticsearch
---


### 聚合基本概念

聚合中最重要的两个概念：

**桶（Buckets）**
满足特定条件的文档的集合

**指标（Metrics）**
对桶内的文档进行统计计算

>每个聚合都是一个或者多个桶和零个或者多个指标的组合。（可理解为给予条件来划分文档，然后在筛选文档）
桶在概念上类似于 SQL 的分组（GROUP BY），而指标则类似于 COUNT() 、 SUM() 、 MAX() 等统计方法。

```json
POST /cars/transactions/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }

GET cars/_mappings
```

<!-- more -->

返回：

```json
{
  "cars": {
    "mappings": {
      "transactions": {
        "properties": {
          "color": {
            "type": "text",     #主需要使用分词
            "fields": {
              "keyword": {       #子需要进行排序或查询
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "make": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "price": {
            "type": "long"
          },
          "sold": {
            "type": "date"
          }
        }
      }
    }
  }
}
```

#### 使用一般的聚合查询：

```json
GET /cars/transactions/_search
{
    "size" : 0,               #定义为0 ，表示不返回搜索结果，只返回聚合的结果
    "aggregations" : {          #聚合操作被置于顶层参数 aggs 之下,可简写 aggs
        "popular_colors" : {             #自定义聚合后的名称
            "terms" : {                 #定义单个桶的类型
              "field" : "color.keyword"
            }
        }
    }
}
```

原理： 这个`terms`桶会为每个碰到的唯一词项动态创建新的桶。 
因为我们告诉它使用**color.kerword**字段，所以**terms**桶会为每个颜色动态创建**新**桶。

返回结果：

```json
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
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "popular_colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4
        },
        {
          "key": "blue",
          "doc_count": 2
        },
        {
          "key": "green",
          "doc_count": 2
        }
      ]
    }
  }
}
```

#### 添加度量指标

计算每种颜色的平均价格：

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color.keyword"
            },
            "aggs": {              #为度量新增 aggs 层
                "avg_price": {            #为度量指定名字
                   "avg": {
                      "field": "price"        #为 price 字段定义 avg 度量
                   }
                }
             }
        }
    }
}
```

>因为子桶中不存在**terms**字段，所以不存在在细分桶的情况，所以**avg_price**与**color.keyword**处于同一层。

返回结果：

```json
{
  ...
  ,
  "aggregations": {
    "popular_colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4,
          "avg_price": {
            "value": 32500
          }
        },
        {
          "key": "blue",
          "doc_count": 2,
          "avg_price": {
            "value": 20000
          }
        },
        {
          "key": "green",
          "doc_count": 2,
          "avg_price": {
            "value": 21000
          }
        }
      ]
    }
  }
}
```

#### 添加嵌套桶指标

案例一：想知道每个颜色的汽车制造商的分布

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color.keyword"
            },
            "aggs": {              
                "avg_price": {            
                   "avg": {
                      "field": "price"        
                   }
                },
                "make": {          #新增了一个聚合
                    "terms": {            #为每个聚合在生成 terms 桶
                        "field": "make.keyword" 
                    }
                }
             }
        }
    }
}
```

返回结果：

```json
{
   ...
   ,
  "aggregations": {
    "popular_colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4,
          "avg_price": {
            "value": 32500
          },
          "make": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "honda",
                "doc_count": 3
              },
              {
                "key": "bmw", 
                "doc_count": 1
              }
            ]
          }
        },
        ....
        ....
      ]
    }
  }
}
```

案例二、为每个颜色的汽车生成商计算最低和最高的价格

```json
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color.keyword"
            },
            "aggs": {              
                "avg_price": {            
                   "avg": {
                      "field": "price"        
                   }
                },
                "make": { 
                    "terms": {
                        "field": "make.keyword" 
                    },
                    "aggs" : {                   #嵌套另一个 aggs 层级
                        "min_price" : { "min": { "field": "price"} },       #最小度量
                        "max_price" : { "max": { "field": "price"} }        #最大度量
                    }
                }
             }
        }
    }
}
```
>分析：每个颜色，所以颜色是一层，且要用terms；一个颜色对应多个厂商，所以厂商要用terms；算的是每个厂商的价格，
所以aggs价格桶与厂商肯定是在一层的。不知道这么说能不能理解，自己体会吧...

#### 条形图

使用 `histogram` 参数:

```json
GET /cars/transactions/_search
{
   "size" : 0,
   "aggs":{
      "price":{
         "histogram":{               #histogram 桶要求两个参数：一个数值字段以及一个定义桶大小间隔
            "field": "price",
            "interval": 20000
         },
         "aggs":{
            "revenue": {
               "sum": {                #嵌套的 sum 度量,计算总价
                 "field" : "price"
               }
             }
         }
      }
   }
}
```

返回结果：

```json
{
...
   "aggregations": {
      "price": {
         "buckets": [
            {
               "key": 0,
               "doc_count": 3,
               "revenue": {
                  "value": 37000
               }
            },
            {
               "key": 20000,
               "doc_count": 4,
               "revenue": {
                  "value": 95000
               }
            },
            {
               "key": 80000,
               "doc_count": 1,
               "revenue": {
                  "value": 80000
               }
            }
         ]
      }
   }
}
```

添加 `extended_stats` 度量，计算一些标准差，平均数，最大最小值等参数:

```json
GET /cars/transactions/_search
{
  "size" : 0,
  "aggs": {
    "makes": {
      "terms": {
        "field": "make.keyword",
        "size": 10
      },
      "aggs": {
        "stats": {
          "extended_stats": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

返回结果：

```json
...
{
    "key": "honda",
    "doc_count": 3,
    "stats": {
        "count": 3,
        "min": 10000,
        "max": 20000,
        "avg": 16666.666666666668,
        "sum": 50000,
        "sum_of_squares": 900000000,
        "variance": 22222222.22222221,
        "std_deviation": 4714.045207910315,
        "std_deviation_bounds": {
            "upper": 26094.757082487296,
            "lower": 7238.5762508460375
          }
        }
}
...
```

#### 时间条形图

```json
GET /cars/transactions/_search
{
   "size" : 0,
   "aggs": {
      "sales": {
         "date_histogram": {
            "field": "sold",
            "interval": "month",         #时间间隔1个月，即每个 bucket 间隔1个月
            "format": "yyyy-MM-dd"       
         }
      }
   }
}
```

返回结果:

```json
{
   ...
   "aggregations": {
   "sales": {
      "buckets": [
        {
          "key_as_string": "2014-01-01",
          "key": 1388534400000,
          "doc_count": 1
        },
        {
          "key_as_string": "2014-02-01",
          "key": 1391212800000,
          "doc_count": 1
        },
        {
          "key_as_string": "2014-03-01",
          "key": 1393632000000,
          "doc_count": 0
        },
        {
          "key_as_string": "2014-04-01",
          "key": 1396310400000,
          "doc_count": 0
        },
        {
          "key_as_string": "2014-05-01",
          "key": 1398902400000,
          "doc_count": 1
        },
        {
          "key_as_string": "2014-06-01",
          "key": 1401580800000,
          "doc_count": 0
        },
        {
          "key_as_string": "2014-07-01",
          "key": 1404172800000,
          "doc_count": 1
        },
        {
          "key_as_string": "2014-08-01",
          "key": 1406851200000,
          "doc_count": 1
        },
        {
          "key_as_string": "2014-09-01",
          "key": 1409529600000,
          "doc_count": 0
        },
        {
          "key_as_string": "2014-10-01",
          "key": 1412121600000,
          "doc_count": 1
        },
        {
          "key_as_string": "2014-11-01",
          "key": 1414800000000,
          "doc_count": 2
        }
      ]
    }
  }
...
}
```

返回结果中有空buckets，如果不想得到，可以通过额外参数来指定：

```json
GET /cars/transactions/_search
{
   "size" : 0,
   "aggs": {
      "sales": {
         "date_histogram": {
            "field": "sold",
            "interval": "month", 
            "format": "yyyy-MM-dd", 
            "min_doc_count" : 1,             #强制返回非空 buckets，默认为0
            "extended_bounds" : {            #扩展的特性，限制时间段
                "min" : "2014-01-01",
                "max" : "2014-12-31"
            }
         }
      }
   }
}
```

真实案例：构建聚合以便按季度展示所有汽车品牌总销售额。
同时按季度、按每个汽车品牌计算销售总额，以便可以找出哪种品牌最赚钱：

>分析：以返回的JSON格式考虑，以季度作为展示，所以季度是X轴，总销售额、季度和品牌销售额属于同一层。
品牌销售额又分为品牌和售销量，所以他们两个应该也处同一层。

```json
GET /cars/transactions/_search
{
   "size" : 0,
   "aggs": {
      "sales": {
         "date_histogram": {
            "field": "sold",
            "interval": "quarter", 
            "format": "yyyy-MM-dd",
            "extended_bounds" : {
                "min" : "2014-01-01",
                "max" : "2014-12-31"
            }
         },
         "aggs": {
            "per_make_sum": {
               "terms": {
                  "field": "make.keyword"
               },
               "aggs": {
                  "sum_price": {
                     "sum": { "field": "price" } 
                  }
               }
            },
            "total_sum": {
               "sum": { "field": "price" } 
            }
         }
      }
   }
}
```

返回的结果：

```json
{
....
 "aggregations": {
    "sales": {
      "buckets": [
        {
          "key_as_string": "2014-01-01",
          "key": 1388534400000,
          "doc_count": 2,
          "per_make_sum": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "bmw",
                "doc_count": 1,
                "sum_price": {
                  "value": 80000
                }
              },
              {
                "key": "ford",
                "doc_count": 1,
                "sum_price": {
                  "value": 25000
                }
              }
            ]
          },
          "total_sum": {
            "value": 105000
          }
        },
...
}
```

