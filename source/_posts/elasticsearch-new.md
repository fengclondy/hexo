---
layout: post
title: ES 6.x新版本特性
date: 2018-06-15 06:13:50
tags: elasticsearch
categories: elasticsearch
---

### 6.3版本新特性

参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.3/sql-getting-started.html

#### 一、支持Sql语句：

|命令|	说明|
|---- | ----- |
|DESC table|	查看该索引的字段和元数据|
|SHOW COLUMNS|	功能同上，只是别名|
|SHOW FUNCTIONS|	列出支持的函数列表，支持通配符过滤|
|SHOW TABLES|	返回索引列表|
|SELECT .. FROM table_name WHERE .. GROUP BY .. HAVING .. ORDER BY .. LIMIT ..|	用来执行查询的命令|

此外，还支持通配符查询，只是通配符目前只支持` %`和 `_`

<!-- more -->

- 1、sql RESTful API返回表格形式数据
```json
POST /_xpack/sql?format=txt
{
  "query": "select * from my_store where price=30"
}
```

- 2、sql转DSL

```json
POST /_xpack/sql/translate
{
  "query": "select * from my_store where price=30"
}
```

>因为 SQL 特性是 xpack 的免费功能，所以是在` _xpack` 这个路径下面，
我们只需要把 SQL 语句传给 query 字段就行了，注意最后面不要加上;结尾，注意是不要！


##### 常见操作命令

```json
## RESTful 的语法
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM twitter"
}

## 获取所有的索引列表
POST /_xpack/sql?format=txt
{
    "query": "SHOW tables"
}

## 名称过滤
POST /_xpack/sql?format=txt
{
    "query": "SHOW TABLES 'twit%'"
}
POST /_xpack/sql?format=txt
{
    "query": "SHOW TABLES 'twitte_'"
}

## 查看该索引的字段和元数据
POST /_xpack/sql?format=txt
{
    "query": "DESC twitter"
}

## 动态生成的字段，查看是否包含某字段
POST /_xpack/sql?format=txt
{
    "query": "SHOW COLUMNS IN twitter"
}

## 查看ES支持哪些函数
POST /_xpack/sql?format=txt
{
  "query": "SHOW FUNCTIONS"
}

## 模糊搜索与过滤别名
POST /_xpack/sql?format=txt
{
    "query": "SELECT SCORE(), * FROM twitter WHERE match(twitter, 'sql is') ORDER BY id DESC"
}
POST /_xpack/sql?format=txt
{
    "query": "SELECT SCORE() as score,name as myname FROM twitter as mytable where name = 'medcl' OR name ='elastic' limit 5"
}
```

>注意点：如果你的索引名称包含横线，如 logstash-201811，只需要做一个用双引号包含，对双引号进行转义即可，如下：
```json
POST /_xpack/sql?format=txt
{
"query":"SELECT COUNT(*) FROM \"logstash-*\""
}
```

>当然，还可以直接使用命令行交互界面,就和sql语句一样：`elasticsearch-6.3.0 ./bin/elasticsearch-sql-cli`

于是乎甚至可以通过JDBC连接elasticsearch。


#### 二、新增汇总统计功能

#### 三、支持java10

#### 四、安全更新