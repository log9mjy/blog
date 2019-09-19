---
title: elasticsearch使用
date: 2018-07-12 11:43:08
tags: 
categories: 搜索引擎
---

#### 安装elasticsearch

elasticsearch下载地址 https://www.elastic.co/downloads/past-releases

安装head插件,提供多种安装方式,查看readme.md有详细描述

https://github.com/mobz/elasticsearch-head

安装ik分词器,注意分词器与elasticsearch的版本要要一致

https://github.com/medcl/elasticsearch-analysis-ik

#### elasticsearch的关键词

集群(cluster):由一个或多个节点组成, 并通过集群名称与其他集群进行区分

节点(node):单个ElasticSearch实例. 通常一个节点运行在一个隔离的容器或虚拟机中

index: 文档通过`index` API被索引——使数据可以被存储和搜索。但是首先我们需要决定文档所在。正如我们讨论的，文档通过其`_index`、`_type`、`_id`唯一确定。我们可以自己提供一个`_id`，或者也使用`index` API 为我们生成一个。

document:通常，我们可以认为对象(object)和文档(document)是等价相通的

type:index存储类型,6.x版本后只能有一种类型

分片(shard):因为ES是个分布式的搜索引擎, 所以索引通常都会分解成不同部分, 而这些分布在不同节点的数据就是分片. ES自动管理和组织分片, 并在必要的时候对分片数据进行再平衡分配, 所以用户基本上不用担心分片的处理细节，一个分片默认最大文档数量是20亿.

副本(replica):ES默认为一个索引创建5个主分片, 并分别为其创建一个副本分片. 也就是说每个索引都由5个主分片成本, 而每个主分片都相应的有一个copy.

对于分布式搜索引擎来说, 分片及副本的分配将是高可用及快速搜索响应的设计核心.主分片与副本都能处理查询请求, 它们的唯一区别在于只有主分片才能处理索引请求.

#### 安装KIbana

https://www.elastic.co/guide/cn/kibana/current/windows.html

#### 数据类型

| 类型           | 表示的数据类型                         |
| -------------- | -------------------------------------- |
| String         | `string`                               |
| Whole number   | `byte`, `short`, `integer`, `long`     |
| Floating point | `float`, `double`                      |
| Boolean        | `boolean`                              |
| Date           | `date`                                 |
| text           | text(用于分词,不能用于聚合)            |
| keyword        | keyword(用于聚合以及排序,不能用于分词) |



#### 查询版本

```javascript
GET  ?pretty
```

响应

```
{
  "name": "node-06",
  "cluster_name": "xb-application",
  "cluster_uuid": "k4nebPYsTNeVHgZop-i4ig",
  "version": {
    "number": "6.3.0",
    "build_flavor": "default",
    "build_type": "tar",
    "build_hash": "424e937",
    "build_date": "2018-06-11T23:38:03.357887Z",
    "build_snapshot": false,
    "lucene_version": "7.3.1",
    "minimum_wire_compatibility_version": "5.6.0",
    "minimum_index_compatibility_version": "5.0.0"
  },
  "tagline": "You Know, for Search"
}
```



#### 创建索引

PUT请求  http://localhost:9200/accounts

返回

```javascript
{
  "acknowledged":true,
  "shards_acknowledged":true
}
```

#### 创建索引并映射

映射字段

```
{
　  "type" : "text", #是数据类型一般文本使用text(可分词进行模糊查询)；keyword无法被分词(不需要执行分词器)，用于精确查找

    "analyzer" : "ik_max_word", #指定分词器，一般使用最大分词：ik_max_word
    
    "normalizer" : "normalizer_name", #字段标准化规则；如把所有字符转为小写；具体如下举例

    "boost" : 1.5, #字段权重；用于查询时评分，关键字段的权重就会高一些，默认都是1；另外查询时可临时指定权重

    "coerce" : true, #清理脏数据：1，字符串会被强制转换为整数 2，浮点数被强制转换为整数；默认为true

    "copy_to" : "field_name", #自定_all字段；指定某几个字段拼接成自定义；具体如下举例

    "doc_values" : true, #加快排序、聚合操作，但需要额外存储空间；默认true，对于确定不需要排序和聚合的字段可false

    "dynamic" : true, #新字段动态添加 true:无限制 false:数据可写入但该字段不保留 'strict':无法写入抛异常

    "enabled" : true, #是否会被索引，但都会存储;可以针对一整个_doc

    "fielddata" : false, #针对text字段加快排序和聚合（doc_values对text无效）；此项官网建议不开启，非常消耗内存

    "eager_global_ordinals": true, #是否开启全局预加载,加快查询；此参数只支持text和keyword，keyword默认可用，而text需要设置fielddata属性
    
    "format" : "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis" ,#格式化 此参数代表可接受的时间格式 3种都接受

    "ignore_above" : 100, #指定字段索引和存储的长度最大值，超过最大值的会被忽略

    "ignore_malformed" : false ,#插入文档时是否忽略类型 默认是false 类型不一致无法插入

    "index_options" : "docs" ,
    # 4个可选参数
    # docs（索引文档号）,
    # freqs（文档号 + 词频），
    # positions（文档号 + 词频 + 位置，通常用来距离查询），
    # offsets（文档号 + 词频 + 位置 + 偏移量，通常被使用在高亮字段）
    # 分词字段默认是position，其他的默认是docs

    "index" : true, #该字段是否会被索引和可查询 默认true

    "fields": {"raw": {"type": "keyword"}} ,#可以对一个字段提供多种索引模式，使用text类型做全文检索，也可使用keyword类型做聚合和排序

    "norms" : true, #用于标准化文档，以便查询时计算文档的相关性。建议不开启

    "null_value" : "NULL", #可以让值为null的字段显式的可索引、可搜索

    "position_increment_gap" : 0 ,#词组查询时可以跨词查询 既可变为分词查询 默认100

    "properties" : {}, #嵌套属性，例如该字段是音乐，音乐还有歌词，类型，歌手等属性

    "search_analyzer" : "ik_max_word" ,#查询分词器;一般情况和analyzer对应
    
    "similarity" : "BM25",#用于指定文档评分模型，参数有三个：
    # BM25 ：ES和Lucene默认的评分模型
    # classic ：TF/IDF评分
    # boolean：布尔模型评分

    "store" : true, #默认情况false,其实并不是真没有存储，_source字段里会保存一份原始文档。
    # 在某些情况下，store参数有意义，比如一个文档里面有title、date和超大的content字段，如果只想获取title和date

    "term_vector" : "no" #默认不存储向量信息，
    # 支持参数yes（term存储），
    # with_positions（term + 位置）,
    # with_offsets（term + 偏移量），
    # with_positions_offsets(term + 位置 + 偏移量)
    # 对快速高亮fast vector highlighter能提升性能，但开启又会加大索引体积，不适合大数据量用
}
```

将text的字段排序或则聚合

在映射时使用

```javascript
"fields": {
            "raw": { 
              "type":  "keyword"
            }
          }
```

排序时

```javascript
 "sort": {
    "city.raw": "asc" 
  },
```

类型上字段以及字段类型

PUT请求  http://localhost:9200/accounts

请求体json:

```bash
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}
```

#### 查询映射

```
GET /my_index/_mapping/my_type
```

```
{
   "library": {
      "mappings": {
         "books": {
            "properties": {
               "name": {
                  "type": "string",
                  "index": "not_analyzed"
               },
               "number": {
                  "type": "object",
                  "dynamic": "true"
               },
               "price": {
                  "type": "double"
               },
               "publish_date": {
                  "type": "date",
                  "format": "dateOptionalTime"
               },
               "title": {
                  "type": "string"
               }
            }
         }
      }
   }
}
```



#### 删除索引/或记录

DELETE请求:     localhost:9200/index名

DELETE请求:     localhost:9200/index名/type名/id

#### 添加记录/更新

PUT请求,需要带上id   localhost:9200/index名/type名/id

请求体

```bash
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理"
}
```

POST请求 ,不需要带ID,系统自动分配ID   localhost:9200/index名/type名

#### 查询

GET请求  /Index/Type/_search或者ID

```js
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "must":     { "match": { "title": "quick" }},
      "must_not": { "match": { "title": "lazy"  }},
      "should": [
                  { "match": { "title": "brown" }},
                  { "match": { "title": "dog"   }}
      ]
    }
  }
}
```

空值查询

```javascript
 "query": {
        "bool": {
            "must_not": {
                "exists": {
                    "field": "enttype_code"
                }
            }
        }
    }
```

非空查询

```javascript
 "query": {
        "bool": {
            "must": {
                "exists": {
                    "field": "enttype_code"
                }
            }
        }
    }
```



范围查询

```java
"query": {
    "bool": {
      
    },
    "range": {
      "FIELD": {
        "gte": 10,
        "lte": 20
      }
    }
  }
```

排序

```javascript
 "sort": [
    {
      "FIELD": {
        "order": "desc"
      }
    }
  ]
```

高亮搜索

```
 "highlight": {
    "fields": {
      "title":{}
    }
  }
```

```
{
        "_index": "book",
        "_type": "chinese",
        "_id": "2",
        "_score": 1.5686159,
        "_source": {
          "id": 2,
          "title": "初中语文",
          "content": "ABCDEFGHIJKLMNOPQRSTUVWXYZ",
          "price": 25.23,
          "createTime": "2019-06-15 12:23:23"
        },
        "highlight": {
          "title": [
            "<em>初中</em><em>语文</em>"
          ]
        }
      },
```

自定义高亮

```
 "highlight": {
    "fields": {
      "title":{
        "pre_tags": [
          "<font color='red'>"
        ],
        "post_tags": [
          "</font>"
        ]
      }
    }
  }
```

pre_tags:前标签

post_tags:后标签

分页

```javascript
"from": 0,
"size": 1
```

聚合

每个activityBrandId的条数

```javascript
{
  "aggs": {
    "bids": {
      "terms": {
        "field": "activityBrandId" 
      }
    }
  }
}
```

```javascript
"aggregations": {
    "bids": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": 0,
          "doc_count": 670
        },
        {
          "key": 1001,
          "doc_count": 4
        },
        {
          "key": 1002,
          "doc_count": 4
        },
        {
          "key": 1000,
          "doc_count": 1
        }
      ]
    }
  }
```

只查询某字段

```javascript
"_source": ["activityBrandId",""]
```

聚合查询

```js
{
  "size": 0,不包含数据,只包含聚合相关
  "aggs": {
    "别名": {
      "min(最小值)": {
        "field": "字段"
      }
    }
  }
}
```



#### 测试分词器

POST请求  localhost:9200/_analyze

```
{
  "analyzer": "ik_max_word",
  "text": "中华人民共和国国歌"
}
```

```
{
    "tokens": [
        {
            "token": "中华人民共和国",
            "start_offset": 0,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "中华人民",
            "start_offset": 0,
            "end_offset": 4,
            "type": "CN_WORD",
            "position": 1
        },
        {
            "token": "中华",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 2
        },
        {
            "token": "华人",
            "start_offset": 1,
            "end_offset": 3,
            "type": "CN_WORD",
            "position": 3
        },
        {
            "token": "人民共和国",
            "start_offset": 2,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 4
        },
        {
            "token": "人民",
            "start_offset": 2,
            "end_offset": 4,
            "type": "CN_WORD",
            "position": 5
        },
        {
            "token": "共和国",
            "start_offset": 4,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 6
        },
        {
            "token": "共和",
            "start_offset": 4,
            "end_offset": 6,
            "type": "CN_WORD",
            "position": 7
        },
        {
            "token": "国",
            "start_offset": 6,
            "end_offset": 7,
            "type": "CN_CHAR",
            "position": 8
        },
        {
            "token": "国歌",
            "start_offset": 7,
            "end_offset": 9,
            "type": "CN_WORD",
            "position": 9
        }
    ]
}
```

1、ik_max_word

会将文本做最细粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为“中华人民共和国、中华人民、中华、华人、人民共和国、人民、共和国、大会堂、大会、会堂等词语。

2、ik_smart

会做最粗粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为中华人民共和国、人民大会堂。

#### springboot整合elasticsearch

##### 1.引入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

##### 2.application.properties

```
spring.data.elasticsearch.cluster-name=elasticsearch
spring.data.elasticsearch.cluster-nodes=127.0.0.1:9300
spring.data.elasticsearch.repositories.enabled=true
```

##### 3.创建实体类

```
@Document(indexName = "goods_index",type = "goods")
public class Goods {
    @Id
    private Integer id;
    @Field(type= FieldType.Text,store = true,analyzer = "ik_max_word")
    private String name;
    @Field(type= FieldType.Text,store = true,analyzer = "ik_max_word")
    private String attrName;
    @Field(type= FieldType.Text,store = true,analyzer = "ik_max_word")
    private String desc;
    @Field(type= FieldType.Integer,store = true,analyzer = "ik_max_word")
    private Integer stock;
    @Field(type= FieldType.Double,store = true,analyzer = "ik_max_word")
    private BigDecimal price;
```

Spring Data通过注解来声明字段的映射属性，有下面的三个注解：

@Document 作用在类，标记实体类为文档对象，一般有两个属性
​	indexName：对应索引库名称
​	type：对应在索引库中的类型
​	shards：分片数量，默认5
​	replicas：副本数量，默认1
@Id 作用在成员变量，标记一个字段作为id主键
@Field 作用在成员变量，标记为文档的字段，并指定字段映射属性：
​	type：字段类型，是枚举：FieldType，可以是text、long、short、date、integer、object等
​		text：存储数据时候，会自动分词，并生成索引
​		keyword：存储数据时候，不会分词建立索引
​		Numerical：数值类型，分两类
​		基本数据类型：long、interger、short、byte、double、float、half_float
​		浮点数的高精度类型：scaled_float需要指定一个精度因子，比如10或100。elasticsearch会把真实值乘以这个因子后存储，取出时再还原。
​		Date：日期类型elasticsearch可以对日期格式化为字符串存储，但是建议我们存储为毫秒值，存储为long，节省空间。
​	index：是否索引，布尔类型，默认是true
​	store：是否存储，布尔类型，默认是false

​	analyzer：分词器名称，这里的ik_max_word即使用ik分词器

日期类型:需要加上注解(使用jackson将对象转为json数据)

```java
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern ="yyyy-MM-dd HH:mm:ss",timezone="GMT+8")
private Date createTime;
```

**在使用时,注解会在生成index时,配置映射**

##### 4.提供template,操作索引,添加索引,删除索引

```
//创建索引
elasticsearchTemplate.createIndex(Goods.class);
//添加隐射
elasticsearchTemplate.putMapping(Goods.class);
```

##### 5.接口继承ElasticsearchRepository,提供一系列对document的操作

```
@Repository
public interface GoodsRepository extends ElasticsearchRepository<Goods,Integer> {
}
```

###### CRUD操作

```
<S extends T> S save(S var1);

<S extends T> Iterable<S> saveAll(Iterable<S> var1);

Optional<T> findById(ID var1);

boolean existsById(ID var1);

Iterable<T> findAll();

Iterable<T> findAllById(Iterable<ID> var1);

long count();

void deleteById(ID var1);

void delete(T var1);

void deleteAll(Iterable<? extends T> var1);

void deleteAll();
```

**自定义方法**

Spring Data 的另一个强大功能，是根据方法名称自动实现功能。在自己的repository中写上该方法

比如：你的方法名叫做：findByTitle，那么它就知道你是根据title查询，然后自动帮你完成，无需写实现类。

当然，方法名称要符合一定的约定：

```
Keyword								Sample
And								findByNameAndPrice
Or								findByNameOrPrice
Is								findByName
Not								findByNameNot
Between							findByPriceBetween
LessThanEqual					findByPriceLessThan
GreaterThanEqual				findByPriceGreaterThan
Before							findByPriceBefore
After							findByPriceAfter
Like							findByNameLike
StartingWith					findByNameStartingWith
EndingWith						findByNameEndingWith
Contains/Containing				findByNameContaining
In								findByNameIn(Collection<String>names)
NotIn							findByNameNotIn(Collection<String>names)
Near							findByStoreNear
True							findByAvailableTrue
False							findByAvailableFalse
OrderBy							findByAvailableTrueOrderByNameDesc
```

###### 条件查询

```
<S extends T> S index(S var1);

Iterable<T> search(QueryBuilder var1);

Page<T> search(QueryBuilder var1, Pageable var2);

Page<T> search(SearchQuery var1);

Page<T> searchSimilar(T var1, String[] var2, Pageable var3);

void refresh();

Class<T> getEntityClass();
```

NativeSearchQueryBuilder：Spring提供的一个查询条件构建器，帮助构建json格式的请求体

```
queryBuilder.withQuery(QueryBuilder queryBuilder);查询条件
queryBuilder.withPageable(Pageable pageable);分页条件
queryBuilder.withSort(SortBuilder sortBuilder);排序条件
repository.search(queryBuilder.build());

```

QueryBuilders：利用QueryBuilders来生成一个查询。QueryBuilders提供了大量的静态方法，用于生成各种不同类型的查询.

```Java
//匹配查询,先分词,在查询
MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("name","手机oppo");
//词条查询,不会分词,直接查询,也可以叫等值查询
TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("stock",100);
 //布尔查询,创建多个查询条件
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
//        must 相当于 与
//        must not 相当于 非
//        should 相当于 或
//        filter  过滤
 boolQueryBuilder.must(matchQueryBuilder);
  boolQueryBuilder.must(termQueryBuilder);
//相似度查询
FuzzyQueryBuilder fuzzyQueryBuilder = QueryBuilders.fuzzyQuery("name","华为手机");
//查询所有
MatchAllQueryBuilder matchAllQueryBuilder = QueryBuilders.matchAllQuery();
//正则表达式查询
RegexpQueryBuilder regexpQueryBuilder = QueryBuilders.regexpQuery("name","");
//模糊查询,* 表示0到多个字符，？表示一个字符
WildcardQueryBuilder wildcardQueryBuilder = QueryBuilders.wildcardQuery("name","*手机*");
//范围查询
RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("stock");
rangeQueryBuilder.gt(100);
rangeQueryBuilder.lt(105);
```

SortBuilders:用于生成排序条件

```
SortBuilders.fieldSort("stock").order(SortOrder.ASC)
```

PageRequest:用于生成分页条件

```
PageRequest.of(0, 10)
```

例子:

```java
//Spring提供的一个查询条件构建器，帮助构建json格式的请求体
NativeSearchQueryBuilder queryBuilder=new NativeSearchQueryBuilder();
//匹配查询,先分词,在查询
MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("name","手机oppo");
//词条查询,不会分词,直接查询
TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("stock",100);
//布尔查询,创建多个查询条件
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
boolQueryBuilder.must(matchQueryBuilder);
boolQueryBuilder.must(termQueryBuilder);
queryBuilder.withQuery(boolQueryBuilder);
queryBuilder.withPageable(PageRequest.of(0, 10));
queryBuilder.withSort(SortBuilders.fieldSort("stock").order(SortOrder.ASC));
Page<Goods> search = goodsRepository.search(queryBuilder.build());
```

###### 高亮查询

默认的DefaultResultMapper不支持高亮展示

重写该方法

```Java
package com.example.elasticsearch.search;

import com.fasterxml.jackson.core.JsonEncoding;
import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonGenerator;
import org.apache.commons.beanutils.PropertyUtils;
import org.elasticsearch.ElasticsearchException;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.get.MultiGetItemResponse;
import org.elasticsearch.action.get.MultiGetResponse;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.common.document.DocumentField;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.ScriptedField;
import org.springframework.data.elasticsearch.core.AbstractResultMapper;
import org.springframework.data.elasticsearch.core.DefaultEntityMapper;
import org.springframework.data.elasticsearch.core.EntityMapper;
import org.springframework.data.elasticsearch.core.aggregation.AggregatedPage;
import org.springframework.data.elasticsearch.core.aggregation.impl.AggregatedPageImpl;
import org.springframework.data.elasticsearch.core.mapping.ElasticsearchPersistentEntity;
import org.springframework.data.elasticsearch.core.mapping.ElasticsearchPersistentProperty;
import org.springframework.data.elasticsearch.core.mapping.SimpleElasticsearchMappingContext;
import org.springframework.data.mapping.context.MappingContext;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;
import org.springframework.util.StringUtils;


import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.nio.charset.Charset;
import java.util.*;

@Component
public class ExtResultMapper extends AbstractResultMapper {

    private final MappingContext<? extends ElasticsearchPersistentEntity<?>, ElasticsearchPersistentProperty> mappingContext;

    public ExtResultMapper() {
        this(new SimpleElasticsearchMappingContext());
    }

    public ExtResultMapper(MappingContext<? extends ElasticsearchPersistentEntity<?>, ElasticsearchPersistentProperty> mappingContext) {

        super(new DefaultEntityMapper(mappingContext));

        Assert.notNull(mappingContext, "MappingContext must not be null!");

        this.mappingContext = mappingContext;
    }

    public ExtResultMapper(EntityMapper entityMapper) {
        this(new SimpleElasticsearchMappingContext(), entityMapper);
    }

    public ExtResultMapper(
            MappingContext<? extends ElasticsearchPersistentEntity<?>, ElasticsearchPersistentProperty> mappingContext,
            EntityMapper entityMapper) {

        super(entityMapper);

        Assert.notNull(mappingContext, "MappingContext must not be null!");

        this.mappingContext = mappingContext;
    }

    @Override
    public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> clazz, Pageable pageable) {

        long totalHits = response.getHits().getTotalHits();
        float maxScore = response.getHits().getMaxScore();

        List<T> results = new ArrayList<>();
        for (SearchHit hit : response.getHits()) {
            if (hit != null) {
                T result = null;
                if (!StringUtils.isEmpty(hit.getSourceAsString())) {
                    result = mapEntity(hit.getSourceAsString(), clazz);
                } else {
                    result = mapEntity(hit.getFields().values(), clazz);
                }

                setPersistentEntityId(result, hit.getId(), clazz);
                setPersistentEntityVersion(result, hit.getVersion(), clazz);
                setPersistentEntityScore(result, hit.getScore(), clazz);

                populateScriptFields(result, hit);

                results.add(result);
            }
        }

        return new AggregatedPageImpl<T>(results, pageable, totalHits, response.getAggregations(), response.getScrollId(),
                maxScore);
    }

    private String concat(Text[] texts) {
        StringBuilder sb = new StringBuilder();
        for (Text text : texts) {
            sb.append(text.toString());
        }
        return sb.toString();
    }


    private <T> void populateScriptFields(T result, SearchHit hit) {
        if (hit.getFields() != null && !hit.getFields().isEmpty() && result != null) {
            for (java.lang.reflect.Field field : result.getClass().getDeclaredFields()) {
                ScriptedField scriptedField = field.getAnnotation(ScriptedField.class);
                if (scriptedField != null) {
                    String name = scriptedField.name().isEmpty() ? field.getName() : scriptedField.name();
                    DocumentField searchHitField = hit.getFields().get(name);
                    if (searchHitField != null) {
                        field.setAccessible(true);
                        try {
                            field.set(result, searchHitField.getValue());
                        } catch (IllegalArgumentException e) {
                            throw new ElasticsearchException(
                                    "failed to set scripted field: " + name + " with value: " + searchHitField.getValue(), e);
                        } catch (IllegalAccessException e) {
                            throw new ElasticsearchException("failed to access scripted field: " + name, e);
                        }
                    }
                }
            }
        }

        for (HighlightField field : hit.getHighlightFields().values()) {
            try {
                PropertyUtils.setProperty(result, field.getName(), concat(field.fragments()));
            } catch (InvocationTargetException | IllegalAccessException | NoSuchMethodException e) {
                throw new ElasticsearchException("failed to set highlighted value for field: " + field.getName()
                        + " with value: " + Arrays.toString(field.getFragments()), e);
            }
        }
    }

    private <T> T mapEntity(Collection<DocumentField> values, Class<T> clazz) {
        return mapEntity(buildJSONFromFields(values), clazz);
    }

    private String buildJSONFromFields(Collection<DocumentField> values) {
        JsonFactory nodeFactory = new JsonFactory();
        try {
            ByteArrayOutputStream stream = new ByteArrayOutputStream();
            JsonGenerator generator = nodeFactory.createGenerator(stream, JsonEncoding.UTF8);
            generator.writeStartObject();
            for (DocumentField value : values) {
                if (value.getValues().size() > 1) {
                    generator.writeArrayFieldStart(value.getName());
                    for (Object val : value.getValues()) {
                        generator.writeObject(val);
                    }
                    generator.writeEndArray();
                } else {
                    generator.writeObjectField(value.getName(), value.getValue());
                }
            }
            generator.writeEndObject();
            generator.flush();
            return new String(stream.toByteArray(), Charset.forName("UTF-8"));
        } catch (IOException e) {
            return null;
        }
    }

    @Override
    public <T> T mapResult(GetResponse response, Class<T> clazz) {
        T result = mapEntity(response.getSourceAsString(), clazz);
        if (result != null) {
            setPersistentEntityId(result, response.getId(), clazz);
            setPersistentEntityVersion(result, response.getVersion(), clazz);
        }
        return result;
    }

    @Override
    public <T> LinkedList<T> mapResults(MultiGetResponse responses, Class<T> clazz) {
        LinkedList<T> list = new LinkedList<>();
        for (MultiGetItemResponse response : responses.getResponses()) {
            if (!response.isFailed() && response.getResponse().isExists()) {
                T result = mapEntity(response.getResponse().getSourceAsString(), clazz);
                setPersistentEntityId(result, response.getResponse().getId(), clazz);
                setPersistentEntityVersion(result, response.getResponse().getVersion(), clazz);
                list.add(result);
            }
        }
        return list;
    }

    private <T> void setPersistentEntityId(T result, String id, Class<T> clazz) {

        if (clazz.isAnnotationPresent(Document.class)) {

            ElasticsearchPersistentEntity<?> persistentEntity = mappingContext.getRequiredPersistentEntity(clazz);
            ElasticsearchPersistentProperty idProperty = persistentEntity.getIdProperty();

            // Only deal with String because ES generated Ids are strings !
            if (idProperty != null && idProperty.getType().isAssignableFrom(String.class)) {
                persistentEntity.getPropertyAccessor(result).setProperty(idProperty, id);
            }
        }
    }

    private <T> void setPersistentEntityVersion(T result, long version, Class<T> clazz) {

        if (clazz.isAnnotationPresent(Document.class)) {

            ElasticsearchPersistentEntity<?> persistentEntity = mappingContext.getPersistentEntity(clazz);
            ElasticsearchPersistentProperty versionProperty = persistentEntity.getVersionProperty();

            // Only deal with Long because ES versions are longs !
            if (versionProperty != null && versionProperty.getType().isAssignableFrom(Long.class)) {
                // check that a version was actually returned in the response, -1 would indicate that
                // a search didn't request the version ids in the response, which would be an issue
                Assert.isTrue(version != -1, "Version in response is -1");
                persistentEntity.getPropertyAccessor(result).setProperty(versionProperty, version);
            }
        }
    }

    private <T> void setPersistentEntityScore(T result, float score, Class<T> clazz) {

        if (clazz.isAnnotationPresent(Document.class)) {

            ElasticsearchPersistentEntity<?> entity = mappingContext.getRequiredPersistentEntity(clazz);

            if (!entity.hasScoreProperty()) {
                return;
            }

            entity.getPropertyAccessor(result) //
                    .setProperty(entity.getScoreProperty(), score);
        }
    }
}
```

```Java
nativeSearchQueryBuilder.withHighlightFields( new HighlightBuilder.Field("title").preTags("<span style=\"color:red\">").postTags("</span>"));

//高亮显示
AggregatedPage<BookIndex> bookIndices = elasticsearchTemplate.queryForPage(searchQuery, BookIndex.class, extResultMapper);
bookIndices.getContent().forEach(System.out::println);
```

###### 聚合查询

cs为别名

minPrice为聚合字段

查询query包含min,max,avg等

```Java
NativeSearchQueryBuilder searchQueryBuilder = new NativeSearchQueryBuilder();
StatsAggregationBuilder statsAggregationBuilder =new StatsAggregationBuilder("cs");
statsAggregationBuilder.field("minPrice");
searchQueryBuilder.withIndices("xw_product")
        .withTypes("type_product")
        .addAggregation(statsAggregationBuilder);
Aggregations query = elasticsearchTemplate.query(searchQueryBuilder.build(), new ResultsExtractor<Aggregations>() {
    @Override
    public Aggregations extract(SearchResponse searchResponse) {
        return searchResponse.getAggregations();
    }
});
```