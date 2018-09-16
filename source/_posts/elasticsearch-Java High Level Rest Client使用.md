---
layout: post
title: ES之Java High Level REST Client使用
date: 2018-03-30 01:11:36
tags: elasticsearch
categories: elasticsearch
---

>本文用到的`_*`只是用来代指某一个值，没有特殊含义。

### 查询

#### 一般的查询

>查询里面必须的两个东西是`SearchRequest`和 `SearchSourceBuilder`，且必须将`SearchSourceBuilder`加入到`SearchRequest`

1. 最简单的查询所有：

```java
SearchRequest searchRequest = new SearchRequest（）;    //定义查询请求
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder（）;   //定义请求体
searchSourceBuilder.query（QueryBuilders.matchAllQuery（））;     //请求体内容为 match_all
searchRequest.source（searchSourceBuilder）;     //将请求体放入查询请求中
```

<!-- more -->

可指定序列和类别

```java
SearchRequest searchRequest = new SearchRequest("_index");   //设置索引
searchRequest.types("_type");      //设置类别
searchRequest.routing("_routing");   //设置路由
searchRequest.indicesOptions(IndicesOptions.lenientExpandOpen());   //如何解析以及如何扩展通配符表达式
searchRequest.preference("_local");   //优先执行本地搜索，默认是整个分片上随机
```

2. term查询

```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder(); 
sourceBuilder.query(QueryBuilders.termQuery("_name", "_value")); 
sourceBuilder.from(0);        //起始位置，默认为0，主要用于分页
sourceBuilder.size(5);        //限制大小，默认为10
sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));    //设置超时时间，60s
```

3. match查询

```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder(); 
MatchQueryBuilder matchQueryBuilder = new MatchQueryBuilder("_name", "_value"); 
matchQueryBuilder.fuzziness(Fuzziness.AUTO);    //启用模糊匹配
matchQueryBuilder.prefixLength(3);       //设置前缀长度
matchQueryBuilder.maxExpansions(10);     //设置最大扩展选项
searchSourceBuilder.query（matchQueryBuilder）;
```
也可以下面方式创建：
```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
QueryBuilder matchQueryBuilder = QueryBuilders.matchQuery（“user”，“kimchy”）
                                                .fuzziness（Fuzziness.AUTO）
                                                .prefixLength（3）
                                                .maxExpansions（10）;
searchSourceBuilder.query（matchQueryBuilder）;
```

4. 指定排序规则

```java
sourceBuilder.sort(new ScoreSortBuilder().order(SortOrder.DESC));       //默认按评分_score降序
//sourceBuilder.sort(new FieldSortBuilder("_uid").order(SortOrder.ASC));    //自定义按_uid升序
```

5. 源过滤
```java
sourceBuilder.fetchSource(false);               //关闭对_source的检索

//用通配符方式的数组来控制哪些字段包含或排除
//String[] includeFields = new String[] {"title", "user", "innerObject.*"};
//String[] excludeFields = new String[] {"_type"};
//sourceBuilder.fetchSource(includeFields, excludeFields);
```

6. 突出显示

```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
HighlightBuilder highlightBuilder = new HighlightBuilder(); 
HighlightBuilder.Field highlightTitle =
        new HighlightBuilder.Field("_title1");     #突出显示字段1
highlightTitle.highlighterType("_type");     #突出显示类型
highlightBuilder.field(highlightTitle);  
HighlightBuilder.Field highlightUser = new HighlightBuilder.Field("_title2");    #突出显示字段2
highlightBuilder.field(highlightUser);
searchSourceBuilder.highlighter(highlightBuilder);
```

7. 聚合

```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
TermsAggregationBuilder aggregation = AggregationBuilders.terms("_aggsName")     
        .field("company.keyword");                      #自定义查询的terms名称和filed字段
aggregation.subAggregation(AggregationBuilders.avg("average_age")
        .field("age"));                                 #自定义子查询的terms名称和filed字段
searchSourceBuilder.aggregation(aggregation);
```

```
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
SuggestionBuilder termSuggestionBuilder =
    SuggestBuilders.termSuggestion("user").text("kmichy"); 
SuggestBuilder suggestBuilder = new SuggestBuilder();
suggestBuilder.addSuggestion("suggest_user", termSuggestionBuilder); 
searchSourceBuilder.suggest(suggestBuilder);
```

下面以一个典型的例子作为演示：
```java
//    "aggs": {
//        "byCategory": {
//            "terms": {
//                "field": "information_type"
//            }
//        },
//        "bySource": {
//            "terms": {
//                "field": "source"
//            }
//        }
//    }

SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
TermsAggregationBuilder categoryAggregationBuilder = AggregationBuilders.terms("byCategory").field("information_type");
TermsAggregationBuilder sourceAggregationBuilder = AggregationBuilders.terms("bySource").field("source");
                
sourceBuilder.aggregation(categoryAggregationBuilder).aggregation(sourceAggregationBuilder);//同层
       //sourceBuilder.aggregation(categoryAggregationBuilder.subAggregation(sourceAggregationBuilder));//嵌套自层

SearchRequest searchRequest = new SearchRequest("_index");
searchRequest.types("_type");
searchRequest.source(sourceBuilder);
```



8. 分析查询与聚合
```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.profile(true);            #可用于简单查询和聚集的分析
```

#### 同步执行

```java
SearchResponse searchResponse = client.search（searchRequest）;
```

#### 异步执行

```java
ActionListener<SearchResponse> listener = new ActionListener<SearchResponse>() {
    @Override
    public void onResponse(SearchResponse searchResponse) {     #执行完成时调用
        
    }

    @Override
    public void onFailure(Exception e) {          #执行失败时调用
        
    }
};

client.searchAsync（searchRequest，listener）;

```

#### 查询响应

1. 响应的状态及类型：

```java
RestStatus status = searchResponse.status();
TimeValue took = searchResponse.getTook();
Boolean terminatedEarly = searchResponse.isTerminatedEarly();
boolean timedOut = searchResponse.isTimedOut();

int totalShards = searchResponse.getTotalShards();
int successfulShards = searchResponse.getSuccessfulShards();
int failedShards = searchResponse.getFailedShards();
for (ShardSearchFailure failure : searchResponse.getShardFailures()) {
    // failures should be handled here
}
```

2. 获取数据：

```java
SearchHits hits = searchResponse.getHits();       #获取响应的hits
long totalHits = hits.getTotalHits();
float maxScore = hits.getMaxScore();

SearchHit[] searchHits = hits.getHits();         #对每一个hits进行轮训操作
for (SearchHit hit : searchHits) {
    // do something with the SearchHit
}
 
String index = hit.getIndex();                 #获取hit的一些信息
String type = hit.getType();
String id = hit.getId();
float score = hit.getScore();

String sourceAsString = hit.getSourceAsString();
Map<String, Object> sourceAsMap = hit.getSourceAsMap();
String documentTitle = (String) sourceAsMap.get("_title");
List<Object> users = (List<Object>) sourceAsMap.get("user");
Map<String, Object> innerObject = (Map<String, Object>) sourceAsMap.get("innerObject");
```

3. 检索结果的突出显示

```java
SearchHits hits = searchResponse.getHits();
for (SearchHit hit : hits.getHits()) {
    Map<String, HighlightField> highlightFields = hit.getHighlightFields();
    HighlightField highlight = highlightFields.get("_title"); 
    Text[] fragments = highlight.fragments();  
    String fragmentString = fragments[0].string();
}
```

4. 检索聚合

```java
Aggregations aggregations = searchResponse.getAggregations();           #从响应里获取聚合
Terms byCompanyAggregation = aggregations.get("_aggsName"); 
Bucket elasticBucket = byCompanyAggregation.getBucketByKey("Elastic"); 
Avg averageAge = elasticBucket.getAggregations().get("average_age"); 
double avg = averageAge.getValue();


Map<String, Aggregation> aggregationMap = aggregations.getAsMap();        #将聚合转换成Map形式
Terms companyAggregation = (Terms) aggregationMap.get("by_company");

List<Aggregation> aggregationList = aggregations.asList();


for (Aggregation agg : aggregations) {                   #轮训处理
    String type = agg.getType();
    if (type.equals(TermsAggregationBuilder.NAME)) {
        Bucket elasticBucket = ((Terms) agg).getBucketByKey("Elastic");
        long numberOfDocs = elasticBucket.getDocCount();
    }
}
```

5. 检索建议

```java
Suggest suggest = searchResponse.getSuggest(); 
TermSuggestion termSuggestion = suggest.getSuggestion("suggest_user"); 
for (TermSuggestion.Entry entry : termSuggestion.getEntries()) { 
    for (TermSuggestion.Entry.Option option : entry) { 
        String suggestText = option.getText().string();
    }
}
```

6. 检索分析结果

```java
Map<String, ProfileShardResult> profilingResults = searchResponse.getProfileResults(); 
for (Map.Entry<String, ProfileShardResult> profilingResult : profilingResults.entrySet()) {  
    String key = profilingResult.getKey(); 
    ProfileShardResult profileShardResult = profilingResult.getValue(); 
}
```