---
layout: post
title: ES进阶
date: 2018-03-22 08:29:47
tags: elasticsearch
categories: elasticsearch
---

### elasticsearch 的版本控制

如果两个线程同时修改一个文档，这时就会发生冲突。

>问题描述：  
比如某件商品存货100件，用户1下单买走1件，剩余99件；
与此同时用户2也下单买走1件，但是用户2不知道用户1已经下单，看到剩余商品仍然是99件。
这样造成系统中显示商品总数比实际数量要多，这种情况在商业系统中肯定是不能容忍的。 

在数据库领域中，有两种方法通常被用来确保并发更新时变更不会丢失：   

1. 悲观并发控制

这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 
一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。  

2. 乐观并发控制

Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 
然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 
例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。

<!-- more -->

#### 方式一、乐观并发控制

当我们之前讨论 index ， GET 和 delete 请求时，我们指出每个文档都有一个 version （版本）号，当文档被修改时版本号递增。 
Elasticsearch 使用这个 version 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。

我们可以利用`_version`号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 version 号来达到这个目的。
如果该版本不是当前版本号，我们的请求将会失败。

```json
PUT /website/blog/1?version=1 
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```

正常返回：

```json
{
  "_index": "website",
  "_type": "blog",
  "_id": "1",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "created": false
}
```

然而，如果我们重新运行相同的索引请求，仍然指定 version=1 ， 
Elasticsearch 返回 409 Conflict HTTP 响应码，和一个如下所示的响应体：

```json
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[blog][1]: version conflict, current version [2] is different than the one provided [1]",
        "index_uuid": "ekHPTnDgRH63lHUpvxqQBA",
        "shard": "3",
        "index": "website"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[blog][1]: version conflict, current version [2] is different than the one provided [1]",
    "index_uuid": "ekHPTnDgRH63lHUpvxqQBA",
    "shard": "3",
    "index": "website"
  },
  "status": 409
}
```

#### 方式二、通过外部系统使用版本控制

一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索， 
这意味着主数据库的所有更改发生时都需要被复制到 Elasticsearch

Elasticsearch 中通过增加 version_type=external 方式指定外部版本号，如果外部版本号是否大于当前文档版本，则可以执行更新操作。

例子：更新一个新的具有外部版本号 5 的博客文章
```json
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```

返回：

```json
{
  "_index": "website",
  "_type": "blog",
  "_id": "2",
  "_version": 5,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "created": true
}
```

当我们再次执行上面的外部版本号更新时，报错。因为外部版本号不大于当前版本号5.

```json
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[blog][2]: version conflict, current version [5] is higher or equal to the one provided [5]",
        "index_uuid": "ekHPTnDgRH63lHUpvxqQBA",
        "shard": "2",
        "index": "website"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[blog][2]: version conflict, current version [5] is higher or equal to the one provided [5]",
    "index_uuid": "ekHPTnDgRH63lHUpvxqQBA",
    "shard": "2",
    "index": "website"
  },
  "status": 409
}
```


### 批量操作

Bulk API与其他的请求体格式稍有不同，如下所示

```json
{ action: { metadata }}
{ request body        }
{ action: { metadata }}
{ request body        }
...
```

这种格式类似一个有效的单行 JSON 文档流，它通过换行符(\n)连接到一起。注意两个要点：

1.每行一定要以换行符(\n)结尾，包括最后一行。这些换行符被用作一个标记，可以有效分隔行。
这些行不能包含未转义的换行符，因为他们将会对解析造成干扰。这意味着这个JSON不能使用`pretty`参数打印。

2.`action/metadata`行指定哪一个文档做什么操作。`metadata`应该指定被索引、创建、更新或者删除的文档的`_index `、` _type`和` _id` 。
`request body`行由文档的`_source`本身组成–文档包含的字段和值。它是更新和创建操作所必需的。


```json
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} }
```

>请注意 delete 动作不能有请求体,它后面跟着的是另外一个操作；同时，谨记最后一个换行符不要落下。

>这个 Elasticsearch 返回值包含 items 数组，这个数组的内容是以请求的顺序列出来的每个请求的结果。

> bulk 请求不是原子的,不能用它来实现事务控制。每个请求是单独处理的，因此一个请求的成功或失败不会影响其他的请求。

### 映射

ElasticSearch中的映射（Mapping）用来定义一个文档，可以定义所包含的字段以及字段的类型、分词器及属性等等。

映射可以分为动态映射和静态映射。 

（1）动态映射 
我们知道，在关系数据库中，需要事先创建数据库，然后在该数据库实例下创建数据表，然后才能在该数据表中插入数据。而ElasticSearch中不需要事先定义映射（Mapping），文档写入ElasticSearch时，会根据文档字段自动识别类型，这种机制称之为动态映射。

（2）静态映射 
当然，在ElasticSearch中也可以事先定义好映射，包含文档的各个字段及其类型等，这种方式称之为静态映射。


#### 动态映射

动态映射可以帮助我们在创建索引后直接将文档数据写入ElasticSearch，让我们尽快享受到ElasticSearch检索功能。
在实际项目中，如果在导入数据前不能确定包含哪些字段或者不方便确定字段类型，可以使用自带的动态映射功能。
当向ElasticSearch写入一个新文档时，需要一个之前没有的字段，会通过动态映射来推断该字段类型。

|JSON数据	|自动推测的类型|
| --------| ----------|
|null	|没有字段被添加|
|true或false	|boolean型|
|小数|	float型|
|数字|	long型|
|日期	|date或text|
|字符串|	text,同时赋值一个field为keyword|
|数组|	由数组第一个非空值决定|
|JSON对象|	object类型|

```
GET book/_mapping
```

#### 静态映射

动态映射的自动类型推测功能并不是100%正确的，这就需要静态映射机制。
静态映射与关系数据库中创建表语句类型，需要事先指定字段类型。
相对于动态映射，静态映射可以添加更加详细字段类型、更精准的配置信息等。    

>在6.x中创建的索引只允许每个索引有单一类型。任何名字都可以用于这个类型，但是只能有且仅有一个。

```json
PUT books
{
  "mappings":{
    "it": {     //it是类名字段，这里表示IT类书籍
       "properties": {
          "bookId": {"type": "long"},
          "bookName": {"type": "text"},
          "publishDate": {"type": "date"}
       }
    }
  }
}
```

返回：

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "books"
}
```

#### 数据类型

支持的所有字段类型：字符串类型（text，keyword）、整数类型、浮点类型、date类型、boolean类型、binary类型（array、object、ip）


#### 元数据

详细的可取官网查看：
https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-fields.html#_document_source_meta_fields

禁用`_source`字段：
```json
PUT my_index
{
  "mappings": {
    "my_type": {
      "_source": {
        "enabled": false
      }
    }
  }
}
```

可以在查询的url中添加`explain`参数，来得到出错的原因。

例子：`GET /website/blog/_validate/query?explain`


### 分析与分析器

#### 原理

分析器 实际上是将三个功能封装到了一个包里：

（1）字符过滤器  
首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 `and`。

（2）分词器  
其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

（3）Token 过滤器  
最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a`， `and`， `the 等无用词），
或者增加词条（例如，像 jump 和 leap 这种同义词）。


#### 使用

当进行全文域搜索时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。然后根据搜索词条，搜索指定的精确值。

>定义分词器的时候可以指定`stopwords`参数，用于删除一些对搜索相关性影响不大的常用词。

>定义分词器的时候可以指定`lowercase`参数，用户转换所有的词汇为小写。

```json
GET /_analyze
{
  "analyzer": "standard",         #默认的分词器
  "text": "Text to analyze"
}
```

返回结果：

```json
{
   "tokens": [                    #实际的结果词条集
      {
         "token":        "text",
         "start_offset": 0,               #开始坐标
         "end_offset":   4,               #结束坐标
         "type":         "<ALPHANUM>",
         "position":     1                #出现的位置
      },
      {
         "token":        "to",
         "start_offset": 5,
         "end_offset":   7,
         "type":         "<ALPHANUM>",
         "position":     2
      },
      {
         "token":        "analyze",
         "start_offset": 8,
         "end_offset":   15,
         "type":         "<ALPHANUM>",
         "position":     3
      }
   ]
}
```

>分析，标准分析器会将"Text to analyze"拆分为三个词条"Text"、"to"、"analyze",然后对这三个词条进行查找。

