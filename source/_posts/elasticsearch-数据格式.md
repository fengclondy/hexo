---
layout: post
title: elasticsearch数据格式
date: 2018-03-29 02:25:16
tags: elasticsearch
categories: elasticsearch
---


### range类型

支持的范围类型：

`integer_range` --- 一系列带符号的32位整数，其范围在 -2^31 ~ 2^31-1

`float_range` --- 一系列单精度32位的浮点值。

`long_range` --- 一系列带符号的64位整数，其范围在 -2^63 ~ 2^63-1

`double_range` --- 一系列双精度64位的浮点值。

`date_range` --- 一系列日期值,无符号64位整数,单位毫秒。

`ip_range` --- 支持IPv4或 IPv6（或混合）地址的一系列ip值。


可接受的参数：

`coerce` --- 尝试将字符串转换为数字并截断整数的分数。接受true（默认）和false。

`boost` --- 映射字段级查询时间提升。接受一个浮点数，默认为 1.0。

`include_in_all` --- 字段值是否应该包含在 all 字段中？接受 true 或 false。默认为 false 。if index 设置为 false，
或者父 object 字段设置 include_in_all 为 false。否则，默认为 true。

`index` --- 该字段是否应该搜索？接受 true（默认）和 false。

`store` --- 字段值是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）。

<!-- more -->


示例：

```json
PUT range_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "expected_attendees": {
          "type": "integer_range"
        },
        "time_frame": {
          "type": "date_range", 
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}

PUT range_index/_doc/1
{
  "expected_attendees" : { 
    "gte" : 10,
    "lte" : 20
  },
  "time_frame" : { 
    "gte" : "2015-10-31 12:00:00", 
    "lte" : "2015-11-01"
  }
}

```

分别对 integer 和 date 字段进行查询：

```json
GET range_index/_search
{
  "query" : {
    "term" : {
      "expected_attendees" : {
        "value": 12
      }
    }
  }
}

GET range_index/_search
{
  "query" : {
    "range" : {
      "time_frame" : { 
        "gte" : "2015-10-31",
        "lte" : "2015-11-01",
        "relation" : "within"         #参数可为 within，contains，intersects（默认）
      }
    }
  }
}
```



### boolean类型

可接受的参数为：

`boost` --- 映射字段级查询时间提升。接受一个浮点数，默认为 1.0。

`doc_values` --- 该字段是否应该以列跨的方式存储在磁盘上，以便以后可以用于排序，聚合或脚本？接受 true （默认）或 false。

`index` --- 该字段是否应该搜索？接受 true（默认）和 false。

`null_value` --- 接受上面列出的任何真值或假值。该值被替换为任何显式 null 值。默认为 null ，这意味着该字段被视为丢失。

`store` --- 字段值是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）。


示例：

```json
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "is_published": {
          "type": "boolean"
        }
      }
    }
  }
}

POST my_index/_doc/1
{
  "is_published": "true" 
}

GET my_index/_search
{
  "query": {
    "term": {
      "is_published": true 
    }
  }
}
```


### date类型

可接受的参数：

`boost` --- 映射字段级查询时间提升。接受一个浮点数，默认为 1.0。

`doc_values` --- 该字段是否应该以列跨的方式存储在磁盘上，以便以后可以用于排序，聚合或脚本？接受 true （默认）或 false。

`format` --- 可以解析的日期格式。默认为 strict_date_optional_time||epoch_millis ,可显示指定为 yyyy-MM-dd HH：mm：ss || yyyy- MM-dd || epoch_millis。

`locale` --- 自从月份以来解析日期时使用的语言环境在所有语言中都没有相同的名称和/或缩写。缺省值是 ROOT 语言环境，

`ignore_malformed` --- 如果 true，格式错误的数字被忽略。如果 false（默认），格式错误的数字会抛出异常并拒绝整个文档。

`index` --- 该字段是否应该搜索？接受 true（默认）和 false。

`null_value` --- 接受一个配置的日期值 format 作为代替任何显式null值的字段。默认为 null，这意味着该字段被视为丢失。

`store` --- 字段值是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）。



示例：

```json
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "date": {
          "type": "date" 
        }
      }
    }
  }
}

PUT my_index/_doc/1
{ "date": "2015-01-01" } 

PUT my_index/_doc/2
{ "date": "2015-01-01T12:10:30Z" } 

PUT my_index/_doc/3
{ "date": 1420070400001 } 

GET my_index/_search
{
  "sort": { "date": "asc"} 
}
```

### text类型

>text类型：支持分词、全文检索，不支持聚合、排序操作。 
适合大字段存储，如：文章详情、content字段等；

可接受的参数：

`analyzer` --- 分析器，其应该被用于 analyzed 既在索引时间和搜索时间（除非被重写字符串字段 search_analyzer）。默认为默认的索引分析器或 standard 分析器。

`boost` --- 映射字段级查询时间提升。接受一个浮点数，默认为 1.0。

`eager_global_ordinals` --- 是否在刷新时加载？接受 true或 false（默认）。对于经常用于（重要）术语聚合的字段，启用此功能是一个好主意。

`fielddata` --- 是否使用内存中的字段数据进行排序，聚合或脚本编写？接受true或false（默认）。

`fielddata_frequency_filter` --- 设置启用时在内存中加载哪些值。默认情况下，所有值都被加载。

`fields` --- 允许为了不同的目的以多种方式对相同的字符串值进行索引，例如搜索字段和用于排序和聚合的多字段，或者由不同分析器分析的相同字符串值。

`index` --- 该字段是否应该搜索？接受 true（默认）或 false。

`index_options` --- 在索引中存储哪些信息，用于搜索和突出显示目的。默认为 positions。 
设置为 docs 可以禁用词频统计及词频位置，这个映射的字段不会计算词的出现次数，对于短语或近似查询也不可用

`norms` --- 评分查询时是否应考虑字段长度。接受 true（默认）或 false。

`position_increment_gap` --- 应插入字符串数组的每个元素之间的假词位置的数量。默认为 position_increment_gap 默认分析仪上的配置 100。

`store` --- 字段值是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）。

`search_analyzer` --- 在搜索时使用的分析器。默认为analyzer设置。

`search_quote_analyzer` --- 在搜索短语时使用的分析器。默认为 search_analyzer 设置。

`similarity` --- 使用哪种评分算法或相似度。默认为 BM25。

`term_vector` --- 术语向量是否存储在一个 analyzed 字段中。默认为 no。 `"term_vector" : "with_positions_offsets"`fvh高亮方式

`norms` --- 文档长度归一值。



### keyWord类型

>keyword类型：支持精确匹配，支持聚合、排序操作。 
适合精准字段匹配，如：url、name、title等字段。
用于索引结构化内容的字段，如电子邮件地址，主机名，状态码，邮政编码或标签。

关键字字段只能按其确切值进行搜索,`terms`查询。


可接受的参数有:

`boost` --- 映射字段级查询时间提升。接受一个浮点数，默认为 1.0。

`doc_values` --- 该字段是否应该以列跨的方式存储在磁盘上，以便以后可以用于排序，聚合或脚本？接受 true （默认）或 false。

`eager_global_ordinals` --- 全球序言是否应该在刷新时急切地加载？接受 true 或 false （默认）。对于经常用于术语聚合的字段，启用此功能是一个不错的主意。

`fields` --- 多字段允许为了不同的目的以多种方式对相同的字符串值进行索引，例如搜索字段和用于排序和聚合的多字段。

`ignore_above` --- 不要索引任何超过此值的字符串。默认为 2147483647所有值都将被接受。

`index` --- 该字段是否应该搜索？接受 true（默认）或 false。

`index_options` --- 为了评分目的，应该在索引中存储哪些信息。默认为，docs但也可以设置为freqs在计算分数时考虑术语频率。

`norms` --- 评分查询时是否应考虑字段长度。接受 true 或 false（默认）。

`null_value` --- 接受替代任何显式 null 值的字符串值。默认为 null，这意味着该字段被视为丢失。

`store` --- 字段值是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）。

`similarity` --- 应使用 哪种评分算法或相似度。默认为 BM25。

`normalizer`  --- 如何在编制索引之前预先处理关键字。默认为 null，意味着关键字保持原样。


示例：

```json
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "tags": {
          "type":  "keyword"
        }
      }
    }
  }
}
```



### ip类型

可接受的参数有：

`boost` --- 映射字段级查询时间提升。接受一个浮点数，默认为 1.0。

`doc_values` --- 该字段是否应该以列跨的方式存储在磁盘上，以便以后可以用于排序，聚合或脚本？接受 true （默认）或 false。

`index` --- 该字段是否应该搜索？接受 true（默认）和 false。

`null_value` --- 接受替代任何显式 null 值的IPv4值。默认为 null，这意味着该字段被视为丢失。

`store` --- 字段值是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）。


示例：

```json
PUT my_index 
{ 
  "mappings"：{ 
    "_doc"：{ 
      "properties"：{ 
        "ip_addr"：{ 
          "type"："ip" 
        } 
      } 
    } 
  } 
} 

PUT my_index / _doc / 1 
{ 
  "ip_addr"："192.168.1.1"
} 

GET my_index / _search 
{ 
  "query"：{ 
    "term"：{ 
      "ip_addr"："192.168.0.0/16" 
    } 
  } 
}
```

对于ipv6类型的，需要使用 "query_string"，同时ipv6地址将需要被转义：

```json
GET my_index/_search
{
  "query": {
    "query_string" : {
      "query": "ip_addr:\"2001:db8::/48\""
    }
  }
}
```

### Geo-point类型

可接受参数：

`ignore_malformed` --- 是否忽略不正确的地理位置。如果 false（默认），格式错误的地理位置引发异常并拒绝整个文档。

示例：

```json
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "location": {
          "type": "geo_point"
        }
      }
    }
  }
}

PUT my_index/_doc/1
{
  "text": "Geo-point as an object",
  "location": { 
    "lat": 41.12,
    "lon": -71.34
  }
}

PUT my_index/_doc/2
{
  "text": "Geo-point as a string",
  "location": "41.12,-71.34" 
}

PUT my_index/_doc/3
{
  "text": "Geo-point as a geohash",
  "location": "drm3btev3e86" 
}

PUT my_index/_doc/4
{
  "text": "Geo-point as an array",
  "location": [ -71.34, 41.12 ] 
}

GET my_index/_search
{
  "query": {
    "geo_bounding_box": { 
      "location": {
        "top_left": {
          "lat": 42,
          "lon": -72
        },
        "bottom_right": {
          "lat": 40,
          "lon": -74
        }
      }
    }
  }
}
```


### Geo-Shape类型

可接受的参数：

`tree` --- 默认 geohash，用于 GeohashPrefixTree（矩形网格，最大24），还有 quadtree，用于 QuadPrefixTree（四叉树，最大50）。

`precision` --- 设置所需的精度，默认 meters，有效距离单位包括：in，inch，yd，yard，mi，miles，km，kilometers， m，meters，cm，centimeters，mm，millimeters。

`tree_levels` --- PrefixTree 使用的最大图层数，默认 50m。只在 Elasticsearch 内部使用。

`strategy` --- 策略参数定义了如何在索引和搜索时间表示形状的方法。有两种策略可用： `recursive`(默认)和`term`。	

`distance_error_pct` --- 用作 PrefixTree 如何精确的提示。默认为0 .025（2.5％），最大支持值为 0.5。注意：如果明确定义了 precision 或 tree_level，则此值将默认为0 。

`orientation` --- 定义如何分析多边形/多面体的顶点顺序。有两种：1.右手规则：right，ccw，counterclockwise，2，左手定则：left，cw，clockwise。默认方向ccw。

`points_only` --- 默认为 false,即 geo_shape 仅为点形状配置字段类型。

`ignore_malformed` -- 是否忽略 GeoJSON 或 WKT 的格式错误，默认为 false。

>`recursive`参数包含 INTERSECTS，DISJOINT，WITHIN，CONTAINS 四钟查询方法，支持所有的多边形状。`terms`支持 INTERSECTS 的点查询。

示例：

```json
PUT / example 
{ 
    "mappings"：{ 
        "doc"：{ 
            "properties"：{ 
                "location"：{ 
                    "type"："geo_shape"，
                    "tree"："quadtree"，
                    "precision"："1m" 
                } 
            } 
        } 
    } 
}
```

现有的关于位置的输入类型与elasticsearch对应关系：


|GeoJSON类型	|WKT类型	|Elasticsearch类型|	描述 |
|-----|-----|------| ------|
|Point  |     POINT  |  point  |   一个地理坐标。|
|LineString | LINESTRING  |  linestring |  给定两个或更多点的任意线。|
|Polygon | POLYGON | polygon | 一个闭合的多边形，其第一个和最后一个点必须匹配，因此需要n + 1顶点创建一个n多边形和最少的4顶点。|
|MultiPoint  | MULTIPOINT | multipoint | 一系列未连接但可能相关的点。|
|MultiLineString  | MULTILINESTRING | multilinestring | 一个单独的线串的数组。|
|MultiPolygon |  MULTIPOLYGON | multipolygon | 一组分离的多边形。|
|GeometryCollection | GEOMETRYCOLLECTION | geometrycollection | 类似于多边形状，但多种类型可以共存（例如Point和LineString）|
|N/A | BBOX| envelope | 通过仅指定左上角和右下角点指定的边界矩形或信封。 |
|N/A |N/A | circle | 一个由中心点指定的圆和带有单位的半径，默认为METERS。|


示例：

```json
POST /example/doc
{
    "location" : {
        "type" : "point",                   #type字段是必须的
        "coordinates" : [-77.03653, 38.897676]       #coordinates也是必须的
    }
}
（GeoJSON格式-上）或者（WKT格式-下）：
POST /example/doc
{
    "location" : "POINT (-77.03653 38.897676)"
}


POST /example/doc
{
    "location" : {
        "type" : "linestring",
        "coordinates" : [[-77.03653, 38.897676], [-77.009051, 38.889939]]
    }
}
（GeoJSON格式-上）或者（WKT格式-下）：
POST /example/doc
{
    "location" : "LINESTRING (-77.03653 38.897676, -77.009051 38.889939)"
}
```

### 数字数据类型

`long` --- 带符号的64位整数，其范围在 -2^63 ~ 2^63-1

`integer` --- 带符号的32位整数，其范围在 -2^31 ~ 2^31-1

`short` --- 带符号的16位整数，其最小值为-32,768，最大值为32,767。

`byte` --- 带符号的8位整数，其最小值为-128，最大值为127。

`double` --- 双精度64位IEEE 754浮点数，限于有限值。

`float` --- 单精度32位IEEE 754浮点数，限于有限值。

`half_float` --- 半精度16位IEEE 754浮点数，限于有限值。

`scaled_float` --- 一个有限的浮点数 long，由固定 double 比例因子缩放。


可接受的参数：

`coerce` --- 尝试将字符串转换为数字并截断整数的分数。接受 true（默认）和 false。

`boost` --- 映射字段级查询时间提升。接受一个浮点数，默认为 1.0。

`doc_values` --- 该字段是否应该以列跨的方式存储在磁盘上，以便以后可以用于排序，聚合或脚本？接受 true （默认）或 false。

`ignore_malformed` --- 如果 true，格式错误的数字被忽略。如果 false（默认），格式错误的数字会抛出异常并拒绝整个文档。

`index` --- 是否应该被搜索？接受 true（默认）和 false。

`null_value` --- 接受与 type 替换任何显式 null 值的字段相同的数字值。默认为 null，这意味着该字段被视为丢失。

`store` --- 字段值是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）


示例：

```json
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "number_of_bytes": {
          "type": "integer"
        },
        "time_in_seconds": {
          "type": "float"
        },
        "price": {
          "type": "scaled_float",
          "scaling_factor": 100
        }
      }
    }
  }
}
```

### token_count类型

类型 token_count 的 integer 字段实际上是一个字段，它接受字符串值，分析它们，然后索引字符串中的标记数量。

接受的参数：

`analyzer` --- 用来分析字符串值的分析器。为获得最佳性能，请使用不带令牌过滤器的分析器。

`enable_position_increments` --- 是否计算位置增量。设置为 false 如果您不想计算分析器过滤器（如 stop）所移除的令牌。默认为 true。

`boost` --- 映射字段级查询时间提升。接受一个浮点数，默认为 1.0。

`doc_values` --- 是否应该以列跨的方式存储在磁盘上，以便以后可以用于排序，聚合或脚本？接受 true （默认）或 false。

`index` --- 是否应该被搜索？接受 true（默认）和 false。

`null_value` --- 接受与 type 替换任何显式 null 值的字段相同的数字值。默认为 null，这意味着该字段被视为丢失。

`store` --- 是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）。


示例：

```json
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "name": { 
          "type": "text",
          "fields": {
            "length": { 
              "type":     "token_count",
              "analyzer": "standard"
            }
          }
        }
      }
    }
  }
}

PUT my_index/_doc/1
{ "name": "John Smith" }

PUT my_index/_doc/2
{ "name": "Rachel Alice Williams" }

GET my_index/_search
{
  "query": {
    "term": {
      "name.length": 3 
    }
  }
}
```

### percolator类型

任何包含 json 对象的字段都可以配置为 percolator 字段。渗滤器字段类型没有设置。
只需配置 percolator 字段类型就足以指示 Elasticsearch 将字段视为查询。

示例：

```json
PUT my_index
{
    "mappings": {
        "_doc": {
            "properties": {
                "query": {
                    "type": "percolator"
                },
                "field": {
                    "type": "text"
                }
            }
        }
    }
}
```

### join类型（6.X新加的）

仍然是一个索引下，借助父子关系，实现类似Mysql中多表关联的操作。

1.每个索引只允许一个Join类型Mapping定义；  

2.父文档和子文档必须在同一个分片上编入索引；这意味着，当进行删除、更新、查找子文档时候需要提供相同的路由值。

3.一个文档可以有多个子文档，但只能有一个父文档。  

4.可以为已经存在的Join类型添加新的关系。

5.当一个文档已经成为父文档后，可以为该文档添加子文档。


设置 qustion 为 answer 的父类：
```json
PUT my_join_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "my_join_field": { 
          "type": "join",
          "relations": {
            "question": "answer" 
          }
        }
      }
    }
  }
}
```

定义父文档：

```json
PUT my_join_index/_doc/1?refresh
{
  "text": "This is a question",
  "my_join_field": "question" 
}

PUT my_join_index/_doc/2?refresh
{
  "text": "This is another question",
  "my_join_field": "question"
}
```

定义子文档：

```json
PUT my_join_index/_doc/3?routing=1&refresh 
{
  "text": "This is an answer",
  "my_join_field": {
    "name": "answer", 
    "parent": "1" 
  }
}

PUT my_join_index/_doc/4?routing=1&refresh
{
  "text": "This is another answer",
  "my_join_field": {
    "name": "answer",
    "parent": "1"
  }
}
```

操作：
1、基于父文档查找子文档

```json
GET my_join_index/_search
{
    "query": {
        "has_parent" : {
            "parent_type" : "question",
            "query" : {
                "match" : {
                    "text" : "This is"
                }
            }
        }
    }
}
```

2、基于子文档查找父文档

```json
GET my_join_index/_search
{
"query": {
        "has_child" : {
            "type" : "answer",
            "query" : {
                "match" : {
                    "text" : "This is question"
                }
            }
        }
    }
}
```

3、聚合操作

```json
GET my_join_index/_search
{
  "query": {
    "parent_id": { 
      "type": "answer",
      "id": "1"
    }
  },
  "aggs": {
    "parents": {
      "terms": {
        "field": "my_join_field#question", 
        "size": 10
      }
    }
  },
  "script_fields": {
    "parent": {
      "script": {
         "source": "doc['my_join_field#question']" 
      }
    }
  }
}
```

4、一对多

要实现的关系，如下图所示：

```

question
    /    \
   /      \
comment  answer
           |
           |
          vote

```

```json
PUT join_multi_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "my_join_field": {
          "type": "join",
          "relations": {
            "question": ["answer", "comment"],  
            "answer": "vote" 
          }
        }
      }
    }
  }
}
```


### nested类型

嵌套对象将数组中的每个对象索引为一个单独的隐藏文档，这意味着每个嵌套对象可以独立于其他 nested 查询来查询。

接受的的参数：

`dynamic` ---是否 properties 应将新动态动态添加到现有的嵌套对象。接受 true（默认）false 和 strict。

`properties` ---嵌套对象中的字段，可以是任何数据类型，包括 nested。可以将新属性添加到现有的嵌套对象。


示例：

```json
GET my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "White" }} 
          ]
        }
      },
      "inner_hits": { 
        "highlight": {
          "fields": {
            "user.first": {}
          }
        }
      }
    }
  }
}
```


### binary类型

可接受的参数：

`doc_values` --- 该字段是否应该以列跨的方式存储在磁盘上，以便以后可以用于排序，聚合或脚本？接受 true （默认）或 false。

`store` --- 字段值是否应该与 source 字段分开存储和检索。接受 true 或 false （默认）。


示例：

```json
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "name": {
          "type": "text"
        },
        "blob": {
          "type": "binary"
        }
      }
    }
  }
}

PUT my_index/my_type/1
{
  "name": "Some binary blob",
  "blob": "U29tZSBiaW5hcnkgYmxvYg==" 
}
```


### 其他

`_index`

`_uid`

`_type`

`_id`

`_source`

`_size`

`_all`

`_field_names`

`_routing`

`_meta`

