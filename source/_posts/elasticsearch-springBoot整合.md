---
layout: post
title: SpringBoot整合ES
date: 2018-03-21 02:22:00
tags: elasticsearch
categories: elasticsearch
---

### 基本的配置

>使用SpringBoot（1.5.10）整合elasticsearch（6.2.2）

1、引入依赖

```xml
<dependency>
	<groupId>org.elasticsearch</groupId>
	<artifactId>elasticsearch</artifactId>
	<version>6.2.2</version>
</dependency>
<!--  es升级需要依赖的 -->
<dependency>
	<groupId>org.elasticsearch.client</groupId>
	<artifactId>elasticsearch-rest-high-level-client</artifactId>
	<version>6.2.2</version>
</dependency>
<dependency>
	<groupId>org.apache.logging.log4j</groupId>
	<artifactId>log4j-core</artifactId>
	<version>2.9.1</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
	<artifactId>log4j-api</artifactId>
	<version>2.9.1</version>
</dependency>
```

<!-- more -->

2、创建rest工厂

```JAVA
import org.apache.http.HttpHost;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.impl.nio.client.HttpAsyncClientBuilder;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import org.elasticsearch.client.RestHighLevelClient;

import java.io.IOException;

public class RestClientFactory {
    private static  String HOST="127.0.0.1";
    private static  int PORT = 9200;
    private static final String SCHEMA = "http";
    private static final int CONNECT_TIME_OUT = 1000;
    private static final int SOCKET_TIME_OUT = 30000;
    private static final int CONNECTION_REQUEST_TIME_OUT = 500;

    private static final int MAX_CONNECT_NUM = 100;
    private static final int MAX_CONNECT_PER_ROUTE = 100;

    private static HttpHost HTTP_HOST=null;
    private static boolean uniqueConnectTimeConfig = false;
    private static boolean uniqueConnectNumConfig = true;
    private static RestClientBuilder builder;
    private static RestClient restClient;
    private static RestHighLevelClient restHighLevelClient;

    static {
        init();
    }

    public static void init(){
        // HOST = Configuration.getProperty("esHost");
        // PORT = Integer.parseInt(Configuration.getProperty("esPort"));
        if(HTTP_HOST==null){
            HTTP_HOST=new HttpHost(HOST,PORT,SCHEMA);
        }
        builder = RestClient.builder(HTTP_HOST);
        if(uniqueConnectTimeConfig){
            setConnectTimeOutConfig();
        }
        if(uniqueConnectNumConfig){
            setMutiConnectConfig();
        }
        restClient = builder.build();
        restHighLevelClient = new RestHighLevelClient(builder);
    }

    /**
     *     主要关于异步httpclient的连接延时配置
     */
    public static void setConnectTimeOutConfig() {
        builder.setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
            @Override
            public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder builder) {
                builder.setConnectTimeout(CONNECT_TIME_OUT);
                builder.setSocketTimeout(SOCKET_TIME_OUT);
                builder.setConnectionRequestTimeout(CONNECTION_REQUEST_TIME_OUT);
                return builder;
            }
        });
    }


    /**
     *    主要关于异步httpclient的连接数配置
     */
     public static void setMutiConnectConfig(){
            builder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
                @Override
                public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpAsyncClientBuilder) {
                    httpAsyncClientBuilder.setMaxConnTotal(MAX_CONNECT_NUM);
                    httpAsyncClientBuilder.setMaxConnPerRoute(MAX_CONNECT_PER_ROUTE);
                    return httpAsyncClientBuilder;
                }
            });
        /*
        builder.setHttpClientConfigCallback(httpClientBuilder -> {
            httpClientBuilder.setMaxConnTotal(MAX_CONNECT_NUM);
            httpClientBuilder.setMaxConnPerRoute(MAX_CONNECT_PER_ROUTE);
            return httpClientBuilder;
        });
        */
     }


    public static RestClient getClient(){
        return restClient;
    }

    public static RestHighLevelClient getHighLevelClient(){
        if(restHighLevelClient==null){
            init();
        }
        return restHighLevelClient;
    }

    public static void close() {
        if (restHighLevelClient != null) {
            try {
                restHighLevelClient.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```

3、



注意:采用  Java High Level REST Client 方式，需进行手动关闭





----------------------------------------------------------

#### 采用Springboot自带的spring-data-start

### springBoot集成elasticsearch

1、导入必备的依赖：

```xml
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

2、编写配置文件

```properties
## 默认为elasticsearch
##spring.data.elasticsearch.cluster-name = elasticsearch
## 配置es节点信息，逗号分隔，如果没有指定，则启动ClientNode
##spring.data.elasticsearch.cluster-nodes= 127.0.0.1
## elasticsearch日志存储目录
spring.data.elasticsearch.properties.path.logs=./elasticsearch/log
## elasticsearch数据存储目录
spring.data.elasticsearch.properties.path.data=./elasticsearch/data       
```

这些配置的属性，最终会设置到`ElasticsearchProperties`这个实体中。


3、创建三个实体  

案例：每个文章(Article)都要属于一个教程(Tutorial),而且每个文章都要有一个作者(Author)。
Tutorial.java  

```java
public class Tutorial implements Serializable{
	private Long id;
	private String name;//教程名称
	
	//setters and getters
	//toString
}
```

Author.java

```java
public class Author implements Serializable{
	/**
	 * 作者id
	 */
	private Long id;
	/**
	 * 作者姓名
	 */
	private String name;
	/**
	 * 作者简介
	 */
	private String remark;
	
	//setters and getters
	//toString
	
}
```

Article.java

```java
public class Article implements Serializable{
    @Id
	private Long id;
	/**标题*/
	private String title;
	/**摘要*/
	private String abstracts;
	/**内容*/
	private String content;
	/**发表时间*/
	private Date postTime;
	/**点击率*/
	private Long clickCount;
	/**作者*/
	private Author author;
	/**所属教程*/
	private Tutorial tutorial;
	
	//setters and getters
	//toString
}
```

4、编写业务逻辑  

因为我们希望Article作为我们文章的搜索入口，所以我们在Article类上添加@Document注解。  
(默认对实体中的所有属性建立索引，可通过@Field注解来进行详细的指定) 

```java
@Document(indexName="projectname",type="article",indexStoreType="fs",shards=5,replicas=1,refreshInterval="-1")
public class Article implements Serializable{
.....
/**发表时间*/
	@Field(format=DateFormat.date_time,index=FieldIndex.no,store=true,type=FieldType.Object)
	private Date postTime;
.....
}
```

@Document注解里面的几个属性，类比mysql的话是这样： index –> DB ；type –> Table ；Document –> row 。  
详细的注解说明：

```java
@Document注解的定义如下：

@Persistent
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
public @interface Document {

String indexName();//索引库的名称，个人建议以项目的名称命名

String type() default "";//类型，个人建议以实体的名称命名

short shards() default 5;//默认分区数

short replicas() default 1;//每个分区默认的备份数

String refreshInterval() default "1s";//刷新间隔

String indexStoreType() default "fs";//索引文件存储类型
}
```

加上了@Document注解之后，默认情况下这个实体中所有的属性都会被建立索引、并且分词。

我们通过@Field注解来进行详细的指定，如果没有特殊需求，那么只需要添加@Document即可。在我们的案例中，使用了@Field针对日期属性postTime上进行了指定。

```java
@Field注解的定义如下：

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Documented
@Inherited
public @interface Field {

FieldType type() default FieldType.Auto;#自动检测属性的类型

FieldIndex index() default FieldIndex.analyzed;#默认情况下分词

DateFormat format() default DateFormat.none;

String pattern() default "";

boolean store() default false;#默认情况下不存储原文

String searchAnalyzer() default "";#指定字段搜索时使用的分词器

String indexAnalyzer() default "";#指定字段建立索引时指定的分词器

String[] ignoreFields() default {};#如果某个字段需要被忽略

boolean includeInParent() default false;
}
```

需要注意的是，这些默认值指的是我们没有在我们没有在属性上添加@Filed注解的默认处理。

一旦添加了@Filed注解，所有的默认值都不再生效。此外，如果添加了@Filed注解，那么type字段必须指定。

5、建立索引类

```java
import com.gqsu.elasticsearch.domain.Article;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

public interface ArticleSearchRepository extends ElasticsearchRepository<Article, Long> {
}

```

6、编写测试案例

(1)创建索引

```java
@Test
	public void testSaveArticleIndex(){
		Author author=new Author();
		author.setId(1L);
		author.setName("tianshouzhi");
		author.setRemark("java developer");

		Tutorial tutorial=new Tutorial();
		tutorial.setId(1L);
		tutorial.setName("elastic search");

		Article article =new Article();
		article.setId(1L);
		article.setTitle("springboot integreate elasticsearch");
		article.setAbstracts("springboot integreate elasticsearch is very easy");
		article.setTutorial(tutorial);
		article.setAuthor(author);
		article.setContent("elasticsearch based on lucene,"
				+ "spring-data-elastichsearch based on elaticsearch"
				+ ",this tutorial tell you how to integrete springboot with spring-data-elasticsearch");
		article.setPostTime(new Date());
		article.setClickCount(1L);

		articleSearchRepository.save(article);
	}
```

(2)查询搜索,输出在控制台

```java
@Test
	public void testSearch(){
		String queryString="springboot";//搜索关键字
		QueryStringQueryBuilder builder=new QueryStringQueryBuilder(queryString);
		Iterable<Article> searchResult = articleSearchRepository.search(builder);
		Iterator<Article> iterator = searchResult.iterator();
		while(iterator.hasNext()){
			System.out.println(iterator.next());
		}
	}
```

说明：以上方式是SpringBoot与ElasticSearch进行本地整合，即将ElasticSearch内嵌在应用，如果我们搭建了ElasticSearch集群，只需要将配置改为如下配置即可：

```properties
## 配置es节点信息，逗号分隔，如果没有指定，则启动ClientNode
spring.data.elasticsearch.cluster-nodes = 115.28.65.149:9300
```

参考教程：https://www.cnblogs.com/ljhdo/p/4887557.html

http://blog.csdn.net/tianyaleixiaowu/article/details/72833940

http://www.tianshouzhi.com/api/tutorials/springboot/101

