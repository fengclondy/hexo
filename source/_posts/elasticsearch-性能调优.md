---
layout: post
title: elasticsearch --- 性能调优
date: 2018-03-28 03:00:32
tags: elasticsearch
categories: elasticsearch
---


### 实时性能

1. 不同分片之间的数据同步是一个很大的花费，默认是1s同步，如果我们不要求实时性，可以执行如下:

```json
PUT twitter 
{
    "settings" : {
        "index" : {
         "refresh_interval":"60s"
        }
    }
}
```

>设置为 -1s ，即不刷新。

<!-- more -->

#### 索引优化

ES索引优化主要从两个方面解决问题：

一、索引数据过程

大家可能会遇到索引数据比较慢的过程。其实明白索引的原理就可以有针对性的进行优化。ES索引的过程到相对Lucene的索引过程多了分布式数据的扩展，而这ES主要是用tranlog进行各节点之间的数据平衡。所以从上我可以通过索引的settings进行第一优化：

这两个参数第一是到tranlog数据达到多少条进行平衡，默认为5000，而这个过程相对而言是比较浪费时间和资源的。所以我们可以将这个值调大一些还是设为-1关闭，进而手动进行tranlog平衡。第二参数是刷新频率，默认为120s是指索引在生命周期内定时刷新，一但有数据进来能refresh像lucene里面commit,我们知道当数据addDoucment后，还不能检索到要commit之后才能行数据的检索，所以可以将其关闭，在最初索引完后手动refresh一之，然后将索引setting里面的index.refresh_interval参数按需求进行修改，从而可以提高索引过程效率。

另外的知道ES索引过程中如果有副本存在，数据也会马上同步到副本中去。我个人建议在索引过程中将副本数设为0，待索引完成后将副本数按需量改回来，这样也可以提高索引效率。

"number_of_replicas": 0

二、检索过程

其实检索速度快度与索引质量有很大的关系。而索引质量的好坏主要与以下几方面有关：

1. 分片数

分片数，与检索速度非常相关的的指标，如果分片数过少或过多都会导致检索比较慢。分片数过多会导致检索时打开比较多的文件别外也会导致多台服务器之间通讯。而分片数过少会导致单个分片索引过大，所以检索速度慢。基于索引分片数=数据总量/单分片数的计算公式，在确定分片数之前需要进行单服务单索引单分片的测试，目前我们测试的结果单个分片的内容为10G。

2. 副本数

副本数与索引的稳定性有比较大的关系，如果Node在非正常挂了，经常会导致分片丢失，为了保证这些数据的完整性，可以通过副本来解决这个问题。建议在建完索引后在执行Optimize后，马上将副本数调整过来。

3. 分词

分词对于索引的影响可大可小，看自己把握。大家或许认为词库越多，分词效果越好，索引质量越好，其实不然。分词有很多算法，大部分基于词表进行分词。也就是说词表的大小决定索引大小。所以分词与索引膨涨率有直接关系。词表不应很多，而对文档相关特征性较强的即可。比如论文的数据进行建索引，分词的词表与论文的特征越相似，词表数量越小，在保证查全查准的情况下，索引的大小可以减少很多。索引大小减少了，那么检索速度也就提高了。

4. 索引段

索引段即lucene中的segments概念，我们知道ES索引过程中会refresh和tranlog也就是说我们在索引过程中segments number不只一个。而segments number与检索是有直接联系的，segments number越多检索越慢，而将segments numbers 有可能的情况下保证为1，这将可以提高将近一半的检索速度。

#### 内存优化

ES对于内存的消耗，和很多因素相关，诸如数据总量、mapping设置、查询方式、查询频度等等。默认的设置虽开箱即用，但不能适用每一种使用场景。作为ES的开发、运维人员，如果不了解ES对内存使用的一些基本原理，就很难针对特有的应用场景，有效的测试、规划和管理集群，从而踩到各种坑，被各种问题挫败。

1. 要理解ES如何使用内存，先要尊重下面两个基本事实：

**ES是JAVA应用**

首先，作为一个JAVA应用，就脱离不开JVM和GC。很多人上手ES的时候，对GC一点概念都没有就去网上抄各种JVM“优化”参数，却仍然被heap不够用，内存溢出这样的问题搞得焦头烂额。即使对于JVM GC机制不够熟悉，头脑里还是需要有这么一个基本概念: 应用层面生成大量长生命周期的对象，是给heap造成压力的主要原因，例如读取一大片数据在内存中进行排序，或者在heap内部建cache缓存大量数据。如果GC释放的空间有限，而应用层面持续大量申请新对象，GC频度就开始上升，同时会消耗掉很多CPU时间。严重时可能恶性循环，导致整个集群停工。因此在使用ES的过程中，要知道哪些设置和操作容易造成以上问题，有针对性的予以规避。

**底层存储引擎是基于Lucene的**

Lucene的倒排索引(Inverted Index)是先在内存里生成，然后定期以段文件(segment file)的形式刷到磁盘的。每个段实际就是一个完整的倒排索引，并且一旦写到磁盘上就不会做修改。 API层面的文档更新和删除实际上是增量写入的一种特殊文档，会保存在新的段里。不变的段文件易于被操作系统cache，热数据几乎等效于内存访问。

基于以上2个基本事实，我们不难理解，为何官方建议的heap size不要超过系统可用内存的一半。heap以外的内存并不会被浪费，操作系统会很开心的利用他们来cache被用读取过的段文件。

Heap分配多少合适？遵从官方建议就没错。 不要超过系统可用内存的一半，并且不要超过32GB。JVM参数呢？对于初级用户来说，并不需要做特别调整，仍然遵从官方的建议，将xms和xmx设置成和heap一样大小，避免动态分配heap size就好了。虽然有针对性的调整JVM参数可以带来些许GC效率的提升，当有一些“坏”用例的时候，这些调整并不会有什么魔法效果帮你减轻heap压力，甚至可能让问题更糟糕。

2. ES的heap是如何被瓜分掉的?

以下分别做解读几个我知道的内存消耗大户：

**Segment Memory**

Segment不是file吗？segment memory又是什么？前面提到过，一个segment是一个完备的lucene倒排索引，而倒排索引是通过词典 (Term Dictionary)到文档列表(Postings List)的映射关系，快速做查询的。 由于词典的size会很大，全部装载到heap里不现实，因此Lucene为词典做了一层前缀索引(Term Index)，这个索引在Lucene4.0以后采用的数据结构是FST (Finite State Transducer)。 这种数据结构占用空间很小，Lucene打开索引的时候将其全量装载到内存中，加快磁盘上词典查询速度的同时减少随机磁盘访问次数。

下面是词典索引和词典主存储之间的一个对应关系图:![](http://p2jr3pegk.bkt.clouddn.com/es01-2.png)


说了这么多，要传达的一个意思就是，ES的data node存储数据并非只是耗费磁盘空间的，为了加速数据的访问，每个segment都有会一些索引数据驻留在heap里。因此segment越多，瓜分掉的heap也越多，并且这部分heap是无法被GC掉的！ 理解这点对于监控和管理集群容量很重要，当一个node的segment memory占用过多的时候，就需要考虑删除、归档数据，或者扩容了。

怎么知道segment memory占用情况呢?  CAT API可以给出答案。

1. 查看一个索引所有segment的memory占用情况： GEt _cat/segments

2. 查看一个node上所有segment占用的memory总和: GEt _cat/nodes


那么有哪些途径减少data node上的segment memory占用呢？ 总结起来有三种方法：

1、删除不用的索引。

2、关闭索引 （文件仍然存在于磁盘，只是释放掉内存）。需要的时候可以重新打开。

3、定期对不再更新的索引做optimize (ES2.0以后更改为force merge api)。这Optimze的实质是对segment file强制做合并，可以节省大量的segment memory。

**Filter Cache**

Filter cache是用来缓存使用过的filter的结果集的，需要注意的是这个缓存也是常驻heap，无法GC的。默认的10% heap size设置工作得够好了，如果实际使用中heap没什么压力的情况下，才考虑加大这个设置。

**Field Data cache**

对搜索结果做排序或者聚合操作，需要将倒排索引里的数据进行解析，然后进行一次倒排。在有大量排序、数据聚合的应用场景，可以说field data cache是性能和稳定性的杀手。
这个过程非常耗费时间，因此ES 2.0以前的版本主要依赖这个cache缓存已经计算过的数据，提升性能。但是由于heap空间有限，当遇到用户对海量数据做计算的时候，就很容易导致heap吃紧，集群频繁GC，根本无法完成计算过程。
ES2.0以后，正式默认启用Doc Values特性(1.x需要手动更改mapping开启)，将field data在indexing time构建在磁盘上，经过一系列优化，可以达到比之前采用field data cache机制更好的性能。因此需要限制对field data cache的使用，最好是完全不用，可以极大释放heap压力。
这里需要注意的是，排序、聚合字段必须为not analyzed。 设想如果有一个字段是analyzed过的，排序的实际对象其实是词典，在数据量很大情况下这种情况非常致命。

**Bulk Queue**

Bulk Queue是做什么用的？当所有的bulk thread都在忙，无法响应新的bulk request的时候，将request在内存里排列起来，然后慢慢清掉。
一般来说，Bulk queue不会消耗很多的heap，但是见过一些用户为了提高bulk的速度，客户端设置了很大的并发量，并且将bulk Queue设置到不可思议的大，比如好几千。这在应对短暂的请求爆发的时候有用，但是如果集群本身索引速度一直跟不上，设置的好几千的queue都满了会是什么状况呢？ 
取决于一个bulk的数据量大小，乘上queue的大小，heap很有可能就不够用，内存溢出了。一般来说官方默认的thread pool设置已经能很好的工作了，建议不要随意去“调优”相关的设置，很多时候都是适得其反的效果。

**Indexing Buffer**

Indexing Buffer是用来缓存新数据，当其满了或者refresh/flush interval到了，就会以segment file的形式写入到磁盘。 这个参数的默认值是10% heap size。根据经验，这个默认值也能够很好的工作，应对很大的索引吞吐量。 
但有些用户认为这个buffer越大吞吐量越高，因此见过有用户将其设置为40%的。到了极端的情况，写入速度很高的时候，40%都被占用，导致OOM。

**Cluster State Buffer**

ES被设计成每个Node都可以响应用户的api请求，因此每个Node的内存里都包含有一份集群状态的拷贝。这个Cluster state包含诸如集群有多少个Node，多少个index，每个index的mapping是什么？有少shard，每个shard的分配情况等等 (ES有各类stats api获取这类数据)。 
在一个规模很大的集群，这个状态信息可能会非常大的，耗用的内存空间就不可忽视了。并且在ES2.0之前的版本，state的更新是由Master Node做完以后全量散播到其他结点的。 频繁的状态更新都有可能给heap带来压力。 在超大规模集群的情况下，
可以考虑分集群并通过tribe node连接做到对用户api的透明，这样可以保证每个集群里的state信息不会膨胀得过大。

**超大搜索聚合结果集的fetch**

ES是分布式搜索引擎，搜索和聚合计算除了在各个data node并行计算以外，还需要将结果返回给汇总节点进行汇总和排序后再返回。无论是搜索，还是聚合，如果返回结果的size设置过大，都会给heap造成很大的压力，特别是数据汇聚节点。超大的size多数情况下都是用户用例不对，
比如本来是想计算cardinality，却用了terms aggregation + size:0这样的方式; 对大结果集做深度分页；一次性拉取全量数据等等。

在开发与维护过程中我们总结出以下优化建议：

1. 尽量运行在Sun/Oracle JDK1.7以上环境中，低版本的jdk容易出现莫名的bug，ES性能体现在在分布式计算中，一个节点是不足以测试出其性能，一个生产系统至少在三个节点以上。

2. 倒排词典的索引需要常驻内存，无法GC，需要监控data node上segment memory增长趋势。

3. 根据机器数，磁盘数，索引大小等硬件环境，根据测试结果，设置最优的分片数和备份数，单个分片最好不超过10GB，定期删除不用的索引，做好冷数据的迁移。

4. 保守配置内存限制参数，尽量使用doc value存储以减少内存消耗，查询时限制size、from参数。

5. 如果不使用_all字段最好关闭这个属性，否则在创建索引和增大索引大小的时候会使用额外更多的CPU，如果你不受限CPU计算能力可以选择压缩文档的_source。这实际上就是整行日志，所以开启压缩可以减小索引大小。

6. 避免返回大量结果集的搜索与聚合。缺失需要大量拉取数据可以采用scan & scroll api来实现。

7. 熟悉各类缓存作用，如field cache, filter cache, indexing cache, bulk queue等等，要设置合理的大小，并且要应该根据最坏的情况来看heap是否够用。

8. 必须结合实际应用场景，并对集群使用情况做持续的监控。


>啥都不说了，先附上大神博客：https://www.felayman.com/articles/2017/11/15/1510726902054.html

完整版学习：https://segmentfault.com/a/1190000011263572

http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html

####  数据索引

ES索引我们可以理解为数据入库的一个过程。我们知道ES是基于Lucene框架的一个分布式检索平台。索引的同样也是基于Lucene创建的，只不过在其上层做了一些封装。
ElasticSearch客户端支持多种语言如PHP、Java、Python、Perl等。


- 常见错误

`SerializationError`：JSON数据序列化出错，通常是因为不支持某个节点值的数据类型

`RequestError`：提交数据格式不正确

`ConflictError`：索引ID冲突

`TransportError`：连接无法建立






### 存储


### 线程池

cache：无限制的线程池，为每个传入的请求创建一个线程。

fixed：一个有着固定大小的线程池，大小由size属性指定，允许你指定一个队列（使用queue_size属性指定）用来保存请求，直到有一个空闲的线程来执行请求。
如果Elasticsearch无法把请求放到队列中（队列满了），该请求将被拒绝。
有很多线程池（可以使用type属性指定要配置的线程类型）。

**最重要的是下面几个**：

index：索引操作。它的类型默认为fixed，size默认为可用处理器的数量，默认为 300。

search：搜索和计数请求。它的类型默认为fixed，size默认为可用处理器的数量乘以3，默认为 1000。

suggest：建议器请求。它的类型默认为fixed，size默认为可用处理器的数量，默认为 1000。

get：实时的GET请求。它的类型默认为fixed，size默认为可用处理器的数量，默认为 1000。

bulk：批量操作。它的类型默认为fixed，size默认为可用处理器的数量，默认为 50。

percolate：预匹配器操作。它的类型默认为fixed，size默认为可用处理器的数量，默认为 1000。

**可以在elasticsearch.properties中可以添加设置**：

```properties
threadpool.index.type: fixed  
threadpool.index.size: 100  
threadpool.index.queue_size: 500  
```

**或者**：

```json
PUT _cluster/settings
{
    "transient": {
        "threadpool.index.type": "fixed",
        "threadpool.index.size": 100,
        "threadpool.index.queue_size": 500
    }
}
```


#### 避免index过大

由于机器资源有限 ，只能想出 将每一天的数据建立一个index+“yyyy-MM-dd” 这样可以有效缓解我们集群的压力

日志的架构是这样的 logstash(client1) 采集日志到 redis  然后通过 logstash(client2) 从redis转至 elasticsearch ，
logstash写入elasticsearch的时候默认就是按照每天来建立索引的 在其配置文件无需指明 index和type 即可。

logstash 自动建立索引的时候是根据格林尼治时间来建立的 正正比我们的时间 迟了8小时，我们需要在logstash的lib里面找到event.rb  然后找到 org.joda.time.DateTimeZone.UTC 格林尼治时间
改成 org.joda.time.DateTimeZone.getDefault() （获取本地时间类型 我这边运行就是中国/上海） 即可


#### 采用G1垃圾回收机制代替默认CMS

 将 elasticsearch.in.sh 内容：

```sh
 # Force the JVM to use IPv4 stack
if [ "x$ES_USE_IPV4" != "x" ]; then
  JAVA_OPTS="$JAVA_OPTS -Djava.net.preferIPv4Stack=true"
fi
 
JAVA_OPTS="$JAVA_OPTS -XX:+UseParNewGC"
JAVA_OPTS="$JAVA_OPTS -XX:+UseConcMarkSweepGC"
 
JAVA_OPTS="$JAVA_OPTS -XX:CMSInitiatingOccupancyFraction=75"
JAVA_OPTS="$JAVA_OPTS -XX:+UseCMSInitiatingOccupancyOnly"
```

替换为：

```sh
JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC"
JAVA_OPTS="$JAVA_OPTS -XX:MaxGCPauseMillis=200"
```

JVM调优，主要目标:1.就是降低 GC 次数时间；2.降低FULLGC 几率


#### 清理没用的缓存

下面方法谨慎操作：

```
http://127.0.0.1:9200/*/_cache/clear  //清除所有索引的cache，如果对查询有实时性要求，慎用！

http://127.0.0.1:9200/*/_optimize    //合并并优化索引
```

参考：https://blog.csdn.net/laoyang360/article/details/79427706


### 问题引入

1. 凤凰网财经版块的新闻数据和评论数据， 1个索引存储采集凤凰网财经版块的新闻数据；1个索引存储相关的财经数据评论结果。

统计： 
1）某条新闻的评论数的多少？ 
2）某条评论属于哪条新闻？ 
3）当前已采集数据的所有评论、评论数汇总，按照评论数逆序排序，以便于图形化展示。


方案一：类似需求，1个索引ifeng_index存储新闻数据；1个索引ifeng_comm_index存储评论数据。 
二者之间通过唯一值建立关联：评论数据中其来源新闻的唯一id值。 
优点：数据分开存储，不存在交叉问题； 
缺点：如果实现需求3），会非常复杂，做全局两通道的遍历和统计。

方案一：类似需求，1个索引ifeng_index存储新闻数据；1个索引ifeng_comm_index存储评论数据。 
二者之间通过唯一值建立关联：评论数据中其来源新闻的唯一id值。 
优点：数据分开存储，不存在交叉问题； 
缺点：如果实现需求3），会非常复杂，做全局两通道的遍历和统计。

参考：https://blog.csdn.net/laoyang360/article/details/79437408

2. 某个词组在Elasitcsearch中的某个document中存在，就一定通过某种匹配方式把它搜出来。

>title=公路局正在治理解放大道路面积水问题。

输入关键词:道路，能否搜索到这个document呢？ 
实际应用中可能需要： 

1）检索关键词”理解”、”解放”、”道路”、“理解放大”，都能搜出这篇文档。

2）单个的字拆分“治”、“水”太多干扰，不要被检索出来。 

3）待检索的词不在词典中，也必须要查到。 

4）待检索词只要在原文title或content中出现，都要检索到。 

5）检索要快，要摒弃wildcard模糊匹配性能问题。

常用的stand标准分词，可以满足要求1）、3）、4）、5）。
样式为：
```json
GET ik_index/_analyze?analyzer=standard
{
   "text":"公路局正在治理解放大道路面积水问题"
}
```
返回：`公,路,局,正,在,治,理,解,放,大,道,路,面,积,水,问,题`

但，会出现冗余数据非常多。 

针对要求2），排除match检索，排除stand分词。 

针对要求5），排除wildcard模糊检索。 

针对要求3）、4)，新词也要被检索到比如：“声临其境”、“孙大剩”等也要能被搜索到。 

针对要求1），采用match_phrase貌似靠谱些。如果检索不出来，采用 match_phrase_prefix

>实际开发中，根据应用场景不同，采用不同的分词器。   
如果选用ik，建议使用 ik_max_word 分词，因为：ik_max_word 的分词结果包含 ik_smart。   
匹配的时候，如果想尽可能的多检索结果，考虑使用 match;   
如果想尽可能精确的匹配分词结果，考虑使用 match_phrase;   
如果短语匹配的时候，怕遗漏，考虑使用 match_phrase_prefix。

参考：https://blog.csdn.net/laoyang360/article/details/79249823


3. 聚合后分页

```json
PUT  book_index
{
    "mappings": {
        "book_type": {
            "properties": {
                "_key": {
                    "type": "keyword",
                    "ignore_above": 256
                },
                "pt": {
                    "type": "date",
                    "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
                },
                "url": {
                    "type": "keyword",
                    "ignore_above": 256
                },
                "title": {
                    "type": "text",
                    "term_vector": "with_positions_offsets",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    },
                    "analyzer": "ik_smart"
                },
                "abstr": {
                    "type": "text",
                    "term_vector": "with_positions_offsets",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    },
                    "analyzer": "ik_smart"
                },
                "rplyinfo": {
                    "type": "text",
                    "term_vector": "with_positions_offsets",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    },
                    "analyzer": "ik_smart"
                },
                "author": {
                    "type": "keyword",
                    "ignore_above": 256
                },
                "booktype": {
                    "type": "keyword",
                    "ignore_above": 256
                },
                "price": {
                    "type": "long"
                }
            }
        }
    }
}


POST book_index/book_type/1
    {
     "title":"《Elasticsearch深入理解》",
     "author":"ERicif",
     "abstr":"Elasticsearch实战书籍",
     "relyinfo":"不错,值得推荐",
     "booktype":"技术",
     "price":79,
     "pt":"2018-04-01"
    }
    
POST book_index/book_type/2
{
  "title":"《大数据之路》",
  "author":"阿里巴巴",
  "abstr":"大数据实现",
  "relyinfo":"不错,值得推荐2",
  "booktype":"技术",
  "price":89,
  "pt":"2018-04-01"
}

POST book_index/book_type/3
{
  "title":"《人性的弱点》",
  "author":"卡耐基",
  "abstr":"直击人性",
  "relyinfo":"不错,值得推荐2",
  "booktype":"励志",
  "price":59,
  "pt":"2018-04-02"
}

POST book_index/book_type/4
{
  "title":"《Flow案例精编》",
  "author":"ERicif",
  "abstr":"Flow案例",
  "relyinfo":"还可以",
  "booktype":"技术",
  "price":57,
  "pt":"2018-04-01"
}

POST book_index/book_type/5
{
  "title":"《kibana案例精编》",
  "author":"ERicif",
  "abstr":"kibana干货",
  "relyinfo":"还可以，不孬",
  "booktype":"技术",
  "price":53,
  "pt":"2018-04-02"
}
```

要求：按照以下条件聚合   
1）相同作者出书量；（聚合）  
2）相同作者的书，取时间最大的返回。（聚合后排序）

```json
Get book_index/_search
{
    "sort": [
        {
            "pt": "desc"
        }
    ],
    "aggs": {
        "count_over_sim": {
            "terms": {
                "field": "author",
                "size": 2147483647,
                "order": {
                    "pt_order": "desc"
                }
            },
            "aggs": {
                "pt_order": {
                    "max": {
                        "field": "pt"
                    }
                }
            }
        }
    },
    "query": {
        "bool": {
            "must": [
                {
                    "bool": {
                        "should": [
                            {
                                "match": {
                                    "booktype": "技术"
                                }
                            }
                        ]
                    }
                },
                {
                    "range": {
                        "pt": {
                            "gte": "2018-04-02",
                            "lte": "2018-04-03"
                        }
                    }
                }
            ]
        }
    },
    "_source": {
        "includes": [
            "title",
            "abstr",
            "pt",
            "booktype",
            "author"
        ]
    },
    "from": 0,
    "size": 10,
    "highlight": {
        "pre_tags": [
            "<span style=\"color:red\">"
        ],
        "post_tags": [
            "</span>"
        ],
        "fields": {
            "title": {}
        }
    }
}
```

>Elasticsearch聚合+分页速度慢,优化方案：改为广度搜索方式。 “collect_mode” : “breadth_first”, 
