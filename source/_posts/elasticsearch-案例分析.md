---
layout: post
title: elasticsearch---案例分析
date: 2018-04-03 01:45:29
tags: elasticsearch
categories: elasticsearch
---

#### 创建索引

```json
PUT demo
{
"mappings":{
    "doc":{
        "properties":{
            "title":{
                "type":"text",
                 "analyzer": "ik_max_word"
            },
            "content":{
                "type":"text",
                 "analyzer": "ik_max_word"
            },
            "uniqueId":{
                "type":"keyword",
                "index":"not_analyzed"
            },
            "created":  {
                "type":   "date", 
                "format": "strict_date_optional_time||epoch_millis"
            }
        }
    }
},
"settings":{
        "number_of_shards":3,
        "number_of_replicas":1
}
}
```

<!-- more -->

关键java代码：

```java
IndexRequest indexRequest = new IndexRequest();
        XContentBuilder builder = JsonXContent.contentBuilder()
                .startObject()
                    .startObject("mappings")
                        .startObject("doc")
                            .startObject("properties")
                                .startObject("title")
                                    .field("type","text")
                                    .field("analyzer","ik_max_word")
                                .endObject()
                                .startObject("content")
                                    .field("type","text")
                                    .field("index","analyzed")
                                    .field("analyzer","ik_max_word")
                                .endObject()
                                .startObject("uniqueId")
                                    .field("type","keyword")
                                    .field("index","not_analyzed")
                                .endObject()
                                .startObject("created")
                                    .field("type","date")
                                    .field("format","strict_date_optional_time||epoch_millis")
                                .endObject()
                            .endObject()
                        .endObject()
                    .endObject()
                    .startObject("settings")
                        .field("number_of_shards",3)
                        .field("number_of_replicas",1)
                    .endObject()
                .endObject();
        indexRequest.source(builder);
            // 生成json字符串
        String source = indexRequest.source().utf8ToString();
        HttpEntity entity = new NStringEntity(source, ContentType.APPLICATION_JSON);
       // 使用RestClient进行操作 而非rhlClient
            Response response = client.performRequest("put", "/demo", Collections.<String, String> emptyMap(),
                entity);
```

#### 创建映射

```json
POST newspaper/sports/_mapping
{
"properties":{
    "content":{
        "type":"text",
        "analyzer":"ik_max_word",
        "index":"analyzed"
    }
}
}
```

关键java代码：

```java
    IndexRequest indexRequest = new IndexRequest();
    XContentBuilder builder = JsonXContent.contentBuilder()
            .startObject()
                 .startObject("properties")
                     .startObject("content")
                         .field("type","text")
                         .field("analyzer","ik_max_word")
                         .field("index","analyzed")
                     .endObject()
                 .endObject()
             .endObject();
    indexRequest.source(builder);
    String source = indexRequest.source().utf8ToString();
    HttpEntity entity = new NStringEntity(source, ContentType.APPLICATION_JSON);
    Response response = client.performRequest("post", "/newspaper/sports/_mapping", Collections.<String, String> emptyMap(),
             entity);
```

#### 关于索引的其他操作

1. 判断索引是否存在

```java
    Response response = restClient.performRequest("HEAD", index);
    boolean exist = response.getStatusLine().getReasonPhrase().equals("OK");
```

2. 删除索引

```java
    Response response = restClient.performRequest("DELETE", indexName);
```




#### 查询文档

实例1：2018年1月26日早八点到晚八点关于费德勒的前十条体育新闻的标题

```json
POST demo/demo/_search
{
    "from":"0",
    "size":"10",
    "_source":["title"],
    "query":{
        "bool":{
            "must":{
                "match":{
                    "title":"费德勒"
                }
            },
            "must":{
                "term":{"tag":"体育"}
            },
            "must":{
                "range":{
                    "publishTime":{
                        "gte":"2018-01-26T08:00:00Z",
                        "lte":"2018-01-26T20:00:00Z"
                    }
                }
            }
        }
    }

}
```

对应的 RestHighLevelClient 方式的关键代码：

```java
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    sourceBuilder.from(0);
    sourceBuilder.size(10);
    sourceBuilder.fetchSource(new String[]{"title"}, new String[]{});
    MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("title", "费德勒");
    TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("tag", "体育");
    RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("publishTime");
    rangeQueryBuilder.gte("2018-01-26T08:00:00Z");
    rangeQueryBuilder.lte("2018-01-26T20:00:00Z");
    BoolQueryBuilder boolBuilder = QueryBuilders.boolQuery();
    boolBuilder.must(matchQueryBuilder);
    boolBuilder.must(termQueryBuilder);
    boolBuilder.must(rangeQueryBuilder);
    sourceBuilder.query(boolBuilder);
    SearchRequest searchRequest = new SearchRequest(index);
    searchRequest.types(type);
    searchRequest.source(sourceBuilder);
```


#### 更新文档

```json
POST demo/demo/2/_update
{
    "doc":{
        "tag":"网球"
    }
}
```

关键java代码：

```java
    UpdateRequest updateRequest = new UpdateRequest(index, type, id);
    Map<String, String> map = new HashMap<>();
    map.put("tag", "网球");
    updateRequest.doc(map);
```

#### 删除文档

1. ID删除方式

```json
DELETE demo/demo/3
```

关键java代码：

```java
    DeleteRequest deleteRequest = new DeleteRequest(_index,_type,_id);
    DeleteResponse response = restHighLevelClient.delete(deleteRequest);
```

2. Delete By Query,即先查询再删除的方式

```json
POST demo/demo/_delete_by_query
{
    "query":{
        "match":{
            "content":"test1"
        }
    }
}
```

关键java代码：

```java
     SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
     sourceBuilder.timeout(new TimeValue(2, TimeUnit.SECONDS));
     TermQueryBuilder termQueryBuilder1 = QueryBuilders.termQuery("content.keyword", _deleteText);
     sourceBuilder.query(termQueryBuilder1);
     SearchRequest searchRequest = new SearchRequest(_index);
     searchRequest.types(_type);
     searchRequest.source(sourceBuilder);
     SearchResponse response = rhlClient.search(searchRequest);
     SearchHits hits = response.getHits();
     List<String> docIds = new ArrayList<>(hits.getHits().length);
     for (SearchHit hit : hits) {
         docIds.add(hit.getId());
     }

     BulkRequest bulkRequest = new BulkRequest();
     for (String id : docIds) {
         DeleteRequest deleteRequest = new DeleteRequest(_index, _type, _id);
         bulkRequest.add(deleteRequest);
     }
     restHighLevelClient.bulk(bulkRequest);
```