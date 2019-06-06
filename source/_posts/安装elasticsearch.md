---
title: elasticsearch使用
date: 2018-07-12 11:43:08
tags: [搜索引擎]
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

**CRUD操作**

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

**条件查询**

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