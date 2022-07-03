# 中间件：ELK学习笔记-5

> 很早之前粗略的学习过Lucene和Elasticsearch，然后最近实习过程中需要用到一些ES的基础操作，虽说使用起来没啥问题，但还是想稍微系统的学习一下，也能在秋招的时候稍微说道说道。
>
> 在 bilibili 上搜了一个课程，课程地址：[黑马：ELK高级搜索](https://www.bilibili.com/video/BV1Nt4y1m7qL)，跟着视频也做了些笔记，总的分了7篇，此为5/7篇。
>
> > 学的很草率...空了再来完善:cry:

## 14. search搜索入门

### 14.1 搜索语法入门

**query string search**

```json
GET /book/_search  // 无条件搜索所有 

// 查询结果：
{
    "took" : 969,         // took：耗费了几毫秒
    "timed_out" : false,  // timed_out：是否超时，这里是没有
    "_shards" : {         // _shards：到几个分片搜索，成功几个，跳过几个，失败几个。 
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits" : {           
        "total" : {        // hits.total：查询结果的数量，3个document
            "value" : 3,
            "relation" : "eq"
        },
        "max_score" : 1.0, // hits.max_score：score的含义，就是document对于一个search的相关度的匹配分数，越相关，就越匹配，分数也高 
        "hits" : [         // hits.hits：包含了匹配搜索的document的所有详细数据
            {
                "_index" : "book",
                "_type" : "_doc",
                "_id" : "1",
                "_score" : 1.0,
                "_source" : {
                    "name" : "Bootstrap开发",
                    "description" : "Bootstrap是由Twitter推出的一个前台页面开发css框架，是一个非常流行的开发框架，此框架集成了多种页面效果。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长css页面开发的程序人员）轻松的实现一个css，不受浏览器限制的精美界面css效果。",
                    "studymodel" : "201002",
                    "price" : 38.6,
                    "timestamp" : "2019-08-25 19:11:35",
                    "pic" : "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
                    "tags" : [
                        "bootstrap",
                        "dev"
                    ]
                }
            },
            {
                "_index" : "book",
                "_type" : "_doc",
                "_id" : "2",
                "_score" : 1.0,
                "_source" : {
                    "name" : "java编程思想",
                    "description" : "java语言是世界第一编程语言，在软件开发领域使用人数最多。",
                    "studymodel" : "201001",
                    "price" : 68.6,
                    "timestamp" : "2019-08-25 19:11:35",
                    "pic" : "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
                    "tags" : [
                        "java",
                        "dev"
                    ]
                }
            },
            {
                "_index" : "book",
                "_type" : "_doc",
                "_id" : "3",
                "_score" : 1.0,
                "_source" : {
                    "name" : "spring开发基础",
                    "description" : "spring 在java领域非常流行，java程序员都在用。",
                    "studymodel" : "201001",
                    "price" : 88.6,
                    "timestamp" : "2019-08-24 19:11:35",
                    "pic" : "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
                    "tags" : [
                        "spring",
                        "java"
                    ]
                }
            }
        ]
    }
}
```

**传参**：与 http 请求传参类似

```json
// 多条件查询
GET /book/_search?q=name:java&sort=price:desc
```

类比SQL： `select * from book where name like ’ %java%’ order by price desc`

```json
{
    "took" : 2,
    "timed_out" : false,
    "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits" : {
        "total" : {
            "value" : 1,
            "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [
            {
                "_index" : "book",
                "_type" : "_doc",
                "_id" : "2",
                "_score" : null,  // _score为空了
                "_source" : {
                    "name" : "java编程思想",
                    "description" : "java语言是世界第一编程语言，在软件开发领域使用人数最多。",
                    "studymodel" : "201001",
                    "price" : 68.6,
                    "timestamp" : "2019-08-25 19:11:35",
                    "pic" : "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
                    "tags" : [
                        "java",
                        "dev"
                    ]
                },
                "sort" : [
                    68.6,
                ]
            }
        ]
    }
}
```

**timeout**：`GET /book/_search?timeout=10ms`

**timeout机制**：指定每个shard只能在给定时间内查询数据，能有几条就返回几条给客户端。

全局设置：配置文件中设置 `search.default_search_timeout：100ms`。默认不超时。

### 14.2 multi-index多索引搜索

**multi-index 搜索模式**：告诉你如何一次性搜索多个 index 和多个 type 下的数据

* `/_search`：所有索引下的所有数据都搜索出来

* `/index1/_search`：指定一个 index，搜索其下所有的数据

* `/index1,index2/_search`：**同时搜索两个index下的数据**

* `/index*/_search`：**按照通配符去匹配多个索引**

应用场景：生产环境 log 索引可以按照日期分开

* `log_to_es_20190910`

* `log_to_es_20190911`

* `log_to_es_20180910`

**简单的搜索原理**

1. 搜索时，存在跨分片的情况，当数据量太大时，所搜十分耗时；
2. 引入timeout机制，指定每个分片只能在给定时间内查询数据，查了多少就返回多少，直接响应给用户。

### 14.3 分页搜索

#### 14.3.1 分页搜索的语法

类似SQL：`select * from book limit 1,5`

```json
// size，from
GET /book/_search?size=10
GET /book/_search?size=10&from=0
GET /book/_search?size=10&from=20
GET /book/_search?from=0&size=3
```

#### 14.3.2 deep paging

**什么是 deep paging**

根据相关度评分倒排序，所以分页过深，协调节点会将大量数据聚合分析。

**deep paging 性能问题**

1. 消耗网络带宽，因为所搜过深的话，各 shard 要把数据传递给 coordinate node，这个过程是有大量数据传递的，消耗网络。

2. 消耗内存，各 shard 要把数据传送给 coordinate node，这个传递回来的数据，是被 coordinate node 保存在内存中的，这样会大量消耗内存。

3. 消耗 cpu，coordinate node 要把传回来的数据进行排序，这个排序过程很消耗 cpu。

所以：鉴于 deep paging 的性能问题，所有应尽量减少使用。

### 14.4 query string基础语法

**query string 基础语法**

```json
GET /book/_search?q=name:java
GET /book/_search?q=+name:java  // 含有+
GET /book/_search?q=-name:java  // 不含有 -
```

一个是掌握 `q=field:search content` 的语法，还有一个是掌握 `+` 和 `-` 的含义

* `+`：搜索含有 `field:search content` 的内容，加不加都一样
* `-`：搜索不含有 `field:search content` 的内容

**_all metadata 的原理和作用**

```json
GET /book/_search?q=java
```

直接可以搜索所有的 field，任意一个 field 包含指定的关键字就可以搜索出来。

我们在进行搜索的时候，难道是对 document 中的每一个 field 都进行一次搜索吗？不是的。

es 中 `_all` 元数据。建立索引的时候，插入一条 docunment，es 会将所有的 field 值进行全量分词，把这些分词，放到 `_all field` 中。在搜索的时候，没有指定 field，就在 `_all` 搜索。

举例：

```json
// 插入一条 document
{
    name:jack
    email:123@qq.com
    address:beijing
}

// 会创建一个 _all field
_all: jack, 123@qq.com, beijing
```

### 14.5 query DSL入门

#### 14.5.1 query DSL概述

query string 后边的参数原来越多，搜索条件越来越复杂，不能满足需求，如：`GET /book/_search?q=name:java&size=10&from=0&sort=price:desc`

**DSL：Domain Specified Language，特定领域的语言**

es 特有的搜索语言，**可在请求体中携带搜索条件**，功能强大。

查询全部： `GET /book/_search`

```json
GET /book/_search
{
  "query": { "match_all": {} }
}
```

排序： `GET /book/_search?q=name:java&sort=price:desc`

```json
GET /book/_search 
{
    "query" : {
        "match" : {
            "name" : " java"
        }
    },
    "sort": [
        { "price": "desc" }
    ]
}
```

分页查询： `GET /book/_search?size=10&from=0`

```json
GET  /book/_search 
{
  "query": { "match_all": {} },
  "from": 0,
  "size": 1
}
```

指定返回字段： `GET /book/_search?_source=name,studymodel`

```json
GET /book/_search 
{
  "query": { "match_all": {} },
  "_source": ["name", "studymodel"]
}
```

通过组合以上各种类型查询，实现复杂查询。

#### 14.5.2 query DSL语法

格式：

```json
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,
        ...
    }
}

{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

举例：

```json
GET /test_index/_search 
{
    "query": {
        "match": {
            "test_field": "test"
        }
    }
}
```

#### 14.5.3 组合多个搜索条件

搜索需求：title 必须包含 elasticsearch，content 可以包含 elasticsearch 也可以不包含，author_id 必须不为111

初始数据：

```json
POST /website/_doc/1
{
    "title": "my hadoop article",
    "content": "hadoop is very bad",
    "author_id": 111
}

POST /website/_doc/2
{
    "title": "my elasticsearch  article",
    "content": "es is very bad",
    "author_id": 112
}
POST /website/_doc/3
{
    "title": "my elasticsearch article",
    "content": "es is very goods",
    "author_id": 111
}
```

搜索：

```json
GET /website/_search
{
    "query": {
        "bool": {
            "must": [  // 必须包含
                {
                    "match": {
                        "title": "elasticsearch"
                    }
                }
            ],
            "should": [  // 应该包含
                {
                    "match": {
                        "content": "elasticsearch"
                    }
                }
            ],
            "must_not": [  // 不能包含
                {
                    "match": {
                        "author_id": 111
                    }
                }
            ]
        }
    }
}
```

返回：

```json
{
    "took" : 1,
    "timed_out" : false,
    "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits" : {
        "total" : {
            "value" : 1,
            "relation" : "eq"
        },
        "max_score" : 0.4700036,
        "hits" : [
            {
                "_index" : "website",
                "_type" : "_doc",
                "_id" : "2",
                "_score" : 0.4700036,
                "_source" : {
                    "title" : "my elasticsearch  article",
                    "content" : "es is very bad",
                    "author_id" : 112
                }
            }
        ]
    }
}
```

更复杂的搜索需求：`select * from test_index where name='tom' or (hired =true and (personality ='good' and rude != true))`

```json
GET /test_index/_search
{
    "query": {
        "bool": {
            "must": { "match":{ "name": "tom" }},
            "should": [
                { "match":{ "hired": true }},
                { "bool": {
                    "must":{ "match": { "personality": "good" }}, 
                    "must_not": { "match": { "rude": true }}
                }}
            ],
            "minimum_should_match": 1
        }
    }
}
```

### 14.6 full-text search全文检索

#### 14.6.1 全文检索

重新创建 book 索引

```json
PUT /book/
{
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "name":{
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            },
            "description":{
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            },
            "studymodel":{
                "type": "keyword"
            },
            "price":{
                "type": "double"
            },
            "timestamp": {
                "type": "date",
                "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
            },
            "pic":{
                "type":"text",
                "index":false
            }
        }
    }
}
```

插入数据

```json
PUT /book/_doc/1
{
    "name": "Bootstrap开发",
    "description": "Bootstrap是由Twitter推出的一个前台页面开发css框架，是一个非常流行的开发框架，此框架集成了多种页面效果。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长css页面开发的程序人员）轻松的实现一个css，不受浏览器限制的精美界面css效果。",
    "studymodel": "201002",
    "price":38.6,
    "timestamp":"2019-08-25 19:11:35",
    "pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
    "tags": [ "bootstrap", "dev"]
}

PUT /book/_doc/2
{
    "name": "java编程思想",
    "description": "java语言是世界第一编程语言，在软件开发领域使用人数最多。",
    "studymodel": "201001",
    "price":68.6,
    "timestamp":"2019-08-25 19:11:35",
    "pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
    "tags": [ "java", "dev"]
}

PUT /book/_doc/3
{
    "name": "spring开发基础",
    "description": "spring 在java领域非常流行，java程序员都在用。",
    "studymodel": "201001",
    "price":88.6,
    "timestamp":"2019-08-24 19:11:35",
    "pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
    "tags": [ "spring", "java"]
}
```

搜索

```json
GET  /book/_search 
{
    "query" : {
        "match" : {
            "description" : "java程序员"
        }
    }
}
```

#### 14.6.2 _score初探

```json
{
    "took" : 1,
    "timed_out" : false,
    "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits" : {
        "total" : {
            "value" : 2,
            "relation" : "eq"
        },
        "max_score" : 2.137549,
        "hits" : [
            {
                "_index" : "book",
                "_type" : "_doc",
                "_id" : "3",
                "_score" : 2.137549,    // sorce更大，先返回
                "_source" : {
                    "name" : "spring开发基础",
                    "description" : "spring 在java领域非常流行，java程序员都在用。",
                    "studymodel" : "201001",
                    "price" : 88.6,
                    "timestamp" : "2019-08-24 19:11:35",
                    "pic" : "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
                    "tags" : [
                        "spring",
                        "java"
                    ]
                }
            },
            {
                "_index" : "book",
                "_type" : "_doc",
                "_id" : "2",
                "_score" : 0.57961315,
                "_source" : {
                    "name" : "java编程思想",
                    "description" : "java语言是世界第一编程语言，在软件开发领域使用人数最多。",
                    "studymodel" : "201001",
                    "price" : 68.6,
                    "timestamp" : "2019-08-25 19:11:35",
                    "pic" : "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
                    "tags" : [
                        "java",
                        "dev"
                    ]
                }
            }
        ]
    }
}
```

**结果分析**

1. 建立索引时：description字段分词，并存入 term 倒排索引

   “java” 关键词在 2，3 文档中都存

   “程序员” 关键词在 2 文档中都存

2. 搜索时，直接找 description 中含有 java 的文档 2，3，并且 3 号文档含有两个 java 字段，一个程序员，所以得分高，排在前面。2 号文档含有一个 java，排在后面。

### 14.7. DSL语法练习

**match_all**

```json
GET /book/_search
{
    "query": {
        "match_all": {}  // 全文检索
    }
}
```

**match**

```json
GET /book/_search
{
	"query": { 
		"match": { 
			"description": "java程序员"  // 限定字段查询
		}
	}
}
```

**multi_match**

```json
GET /book/_search
{
    "query": {
        "multi_match": {
            "query": "java程序员",
            "fields": ["name", "description"]  // 多field查询
        }
    }
}
```

**range query**

```json
GET /book/_search
{
    "query": {
        "range": {  // 范围查询
            "price": {
                "gte": 80,
                "lte": 90
            }
        }
    }
}
```

**term query**

字段为 keyword 时，存储和搜索都不分词

```json
GET /book/_search
{
    "query": {
        "term": {
            "description": "java程序员"
        }
    }
}
```

**terms query**

```json
GET /book/_search
{
    "query": { 
        "terms": { 
            "description": [ "java", "程序员"] 
        }
    }
}
```

**exist query**

```json
GET /_search
{
    "query": {
        "exists": {  // 查询有某些字段值的文档，比如以下为查询有join_date字段的文档
            "field": "join_date"
        }
    }
}
```

**Fuzzy query**

返回包含与搜索词类似的词的文档，该词由 Levenshtein 编辑距离度量。

包括以下几种情况：

- 更改角色（box→fox）

- 删除字符（aple→apple）

- 插入字符（sick→sic）

- 调换两个相邻字符（ACT→CAT） 


```json
GET /book/_search
{
    "query": {
        "fuzzy": {
            "description": {
                "value": "jave"
            }
        }
    }
}
```

**IDs / 根据 ID 查询**

```json
GET /book/_search
{
    "query": {
        "ids" : {
            "values" : ["1", "4", "100"]
        }
    }
}
```

**prefix / 前缀查询**

```json
GET /book/_search
{
    "query": {
        "prefix": {
            "description": {
                "value": "spring"
            }
        }
    }
}
```

**regexp query / 正则查询** 

```json
GET /book/_search
{
    "query": {
        "regexp": {
            "description": {
                "value": "j.*a",
                "flags" : "ALL",
                "max_determinized_states": 10000,
                "rewrite": "constant_score"
            }
        }
    }
}
```

### 14.8 Filter

**filter与query示例**

需求：用户查询 description 中有 "java程序员"，并且价格大于80小于90的数据。

* 仅适用query：

```json
GET /book/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "description": "java程序员"
                    }
                },
                {
                    "range": {
                        "price": {
                            "gte": 80,
                            "lte": 90
                        }
                    }
                }
            ]
        }
    }
}
```

* 使用filter:

```json
GET /book/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "description": "java程序员"
                    }
                }
            ],
            "filter": {  // 过滤器
                "range": {
                    "price": {
                        "gte": 80,
                        "lte": 90
                    }
                }
            }
        }
    }
}
```

**filter 与 query 概念对比：**

* filter：仅仅只是按照搜索条件过滤出需要的数据而已，不计算任何相关度分数，**对相关度没有任何影响**。

* query：会去计算每个document相对于搜索条件的相关度，并按照相关度进行排序。

**应用场景：**

* 一般来说，如果你是在进行搜索，需要将最匹配搜索条件的数据先返回，那么用 query；

* 如果你只是要根据一些条件筛选出一部分数据，不关注其排序，那么用 filter

**filter 与 query 性能对比：**

* filter：不需要计算相关度分数，不需要按照相关度分数进行排序，同时还有内置的自动 cache 最常使用 filter 的数据

* query：相反，要计算相关度分数，按照分数进行排序，而且无法cache结果

### 14.9 定位错误语法

验证错误语句：

```json
GET /book/_validate/query?explain
{
    "query": {
        "mach": {
            "description": "java程序员"
        }
    }
}

// 返回：
{
  "valid" : false,
  "error" : "org.elasticsearch.common.ParsingException: no [query] registered for [mach]"
}
```

正确的查询语句：

```json
GET /book/_validate/query?explain
{
    "query": {
        "match": {
            "description": "java程序员"
        }
    }
}

// 返回：
{
    "_shards" : {
        "total" : 1,
        "successful" : 1,
        "failed" : 0
    },
    "valid" : true,
    "explanations" : [  // 查询计划
        {
            "index" : "book",
            "valid" : true,
            "explanation" : "description:java description:程序员"
        }
    ]
}
```

一般用在那种特别复杂庞大的搜索下，比如你一下子写了上百行的搜索，这个时候可以先用 validate api 去验证一下，搜索是否合法。

合法以后，explain 就像 mysql 的执行计划，可以看到搜索的目标等信息。

### 14.10 定制排序规则

#### 14.10.1 默认排序规则

默认情况下，是**按照 _score 降序排序的**

```json
GET book/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "description": "java程序员"
                    }
                }
            ]
        }
    }
}
```

然而，某些情况下，可能没有有用的 _score，比如说 filter

以下简写会报错

```json
GET book/_search
{
    "query": {
        "filter":{
            "term" : {
                "studymodel" : "201001"
            }
        }
    }
}
```

正确写法：使用 constant_score

```json
GET /book/_search 
{
    "query": {
        "constant_score": {
            "filter" : {
                "term" : {
                    "studymodel" : "201001"
                }
            }
        }
    }
}
```

#### 14.10.2 定制排序规则

相当于 SQL 中 order by，`?sort=sprice:desc`

```json
GET /book/_search 
{
    "query": {
        "constant_score": {
            "filter" : {
                "term" : {
                    "studymodel" : "201001"
                }
            }
        }
    },
    "sort": [
        {
            "price": {
                "order": "asc"
            }
        }
    ]
}
```

#### 14.10.3 text字段排序问题

场景：数据库中按照某个字段排序，SQL 只需写 order by "字段名" 即可。

而在 es 中如果对一个 text field 进行排序，结果往往不准确，因为分词后是多个单词，再排序就不是我们想要的结果了。

通常解决方案是：

* 方案1：在设置 mapping 时，令 `fielddata:ture`，会按照第一个分词进行字典序的排列，结果往往不准确
* 方案2：**将一个 text field 建立两次索引，一个分词，用来进行搜索；一个不分词，用来进行排序**。

```json
PUT /website 
{
    "mappings": {
        "properties": {
            "title": {  // 一个text类型的一个keyword类型的
                "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword"
                    }        
                }      
            },
            "content": {
                "type": "text"
            },
            "post_date": {
                "type": "date"
            },
            "author_id": {
                "type": "long"
            }
        }
    }
}
```

插入数据

```json
PUT /website/_doc/1
{
    "title": "first article",
    "content": "this is my second article",
    "post_date": "2019-01-01",
    "author_id": 110
}

PUT /website/_doc/2
{
    "title": "second article",
    "content": "this is my second article",
    "post_date": "2019-01-01",
    "author_id": 110
}

PUT /website/_doc/3
{
    "title": "third article",
    "content": "this is my third article",
    "post_date": "2019-01-02",
    "author_id": 110
}
```

搜索

```json
GET /website/_search
{
    "query": {
        "match_all": {}
    },
    "sort": [
        {
            "title.keyword": {
                "order": "desc"
            }
        }
    ]
}
```

### 14.11 scroll分批查询

场景：下载某一个索引中1亿条数据到文件或是数据库。

首先：不能一下全查出来，系统内存溢出，所以使用 **scoll 滚动搜索技术**，一批一批查询。

scoll 搜索会在第一次搜索的时候，**保存一个当时的视图快照**，之后只会基于该旧的视图快照提供数据搜索，如果这个期间数据变更，是不会让用户看到的

每次发送 scroll 请求，我们还需要指定一个 scoll 参数，指定一个**时间窗口**，每次搜索请求只要在这个时间窗口内能完成就可以了。

搜索：1m为1分钟，滚动查询条数

```json
GET /book/_search?scroll=1m
{
    "query": {
        "match_all": {}
    },
    "size": 3
}

// 返回
{
    "_scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAMOkWTURBNDUtcjZTVUdKMFp5cXloVElOQQ==",
    "took" : 3,
    "timed_out" : false,
    "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits" : {
        "total" : {
            "value" : 3,
            "relation" : "eq"
        },
        "max_score" : 1.0,
        "hits" : [
            // ...
        ]
    }
}
```

获得的结果会有一个 scoll_id，下一次再发送 scoll 请求的时候，必须带上这个 scoll_id

```json
GET /_search/scroll
{
    "scroll": "1m", 
    "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAMOkWTURBNDUtcjZTVUdKMFp5cXloVElOQQ=="
}
```

**与分页区别**：

* 分页给用户看的  deep paging

* scroll 是用户系统内部操作，如下载批量数据，数据转移，零停机改变索引映射。

## 15. Java API实现搜索

### 15.1 查询全部

```java
@SpringBootTest(classes = SearchApplication.class)
@RunWith(SpringRunner.class)
public class TestSearch {

    @Autowired
    RestHighLevelClient client;

    /**
     * 查询所有
     * @throws IOException
     */
    @Test
    public void testSearchAll() throws IOException {

        // 1.构建搜索请求
        SearchRequest searchRequest = new SearchRequest("book");
        // 1.1 构建查询内容
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());
        // 1.2 获取某些字段
        searchSourceBuilder.fetchSource(new String[]{"name", "description", "price"}, new String[]{});
        // 1.3 封装
        searchRequest.source(searchSourceBuilder);

        // 2.执行搜索
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        // 3.获取结果
        SearchHits hits = searchResponse.getHits();
        for (SearchHit hit : hits) {
            String id = hit.getId();
            float score = hit.getScore();
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            System.out.println("id:" + id);
            System.out.println("scores:" + score);
            System.out.println("name:" + (String) sourceAsMap.get("name"));
            System.out.println("description:" + (String) sourceAsMap.get("description"));
            System.out.println("price:" + sourceAsMap.get("price"));
            System.out.println("-------------------");
        }
    }
}
```

### 15.2 分页搜索

```java
/**
 * 分页查询
 * @throws IOException
 */
@Test
public void testSearchPage() throws IOException {
    // 1.构建搜索请求
    SearchRequest searchRequest = new SearchRequest("book");
    // 1.1 构建查询内容
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.matchAllQuery());
    // 1.2 获取某些字段
    searchSourceBuilder.from(1);
    searchSourceBuilder.size(2);
    // 1.3 封装
    searchRequest.source(searchSourceBuilder);

    // 2.执行搜索
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    // 3.获取结果
    SearchHits hits = searchResponse.getHits();
    for (SearchHit hit : hits) {
        String id = hit.getId();
        float score = hit.getScore();
        Map<String, Object> sourceAsMap = hit.getSourceAsMap();
        System.out.println("id:" + id);
        System.out.println("scores:" + score);
        System.out.println("name:" + (String) sourceAsMap.get("name"));
        System.out.println("description:" + (String) sourceAsMap.get("description"));
        System.out.println("price:" + sourceAsMap.get("price"));
        System.out.println("-------------------");
    }
}
```

### 15.3 ID 搜索

```java
/**
 * ids搜索
 * @throws IOException
 */
@Test
public void testSearchIDs() throws IOException {

    // 1.构建搜索请求
    SearchRequest searchRequest = new SearchRequest("book");
    // 1.1 构建查询内容
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.idsQuery().addIds("1", "2", "100"));
    // 1.3 封装
    searchRequest.source(searchSourceBuilder);

    // 2.执行搜索
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    // 3.获取结果
    SearchHits hits = searchResponse.getHits();
    for (SearchHit hit : hits) {
        String id = hit.getId();
        float score = hit.getScore();
        Map<String, Object> sourceAsMap = hit.getSourceAsMap();
        System.out.println("id:" + id);
        System.out.println("scores:" + score);
        System.out.println("name:" + (String) sourceAsMap.get("name"));
        System.out.println("description:" + (String) sourceAsMap.get("description"));
        System.out.println("price:" + sourceAsMap.get("price"));
        System.out.println("-------------------");
    }
}
```

### 15.4 按关键词搜索

```java
searchSourceBuilder.query(QueryBuilders.matchQuery("description", "java程序员"));
```

### 15.5 按 term 搜索

```java
searchSourceBuilder.query(QueryBuilders.termQuery("description", "java程序员"));
```

### 15.6 按 multi match 搜索

```java
searchSourceBuilder.query(QueryBuilders.multiMatchQuery("java程序员", "name", "description"));
```

### 15.7 按 bool query 搜索

```java
// 按boolquery搜索
// 构建multimatch
MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("java程序员", "name", "description");
// 构建match请求
MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("studymodel", "201001");
// 构建bool请求
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
boolQueryBuilder.must(multiMatchQueryBuilder);
boolQueryBuilder.should(matchQueryBuilder);

// 将bool qury放入 searchSourceBuilder
searchSourceBuilder.query(boolQueryBuilder);
```

### 15.8 filter 搜索

```java
// 前面构造的bool query和上面相同
boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gt(10).lt(90));
searchSourceBuilder.query(boolQueryBuilder);
```

### 15.9 sort搜索

```java
searchSourceBuilder.sort("price", SortOrder.ASC);
```

## 16 评分机制详解

### 16.1 评分机制TF\IDF

#### 16.1.1 算法介绍

relevance score 算法，简单来说，就是计算出一个索引中的文本与搜索文本他们之间的关联匹配程度。

Elasticsearch 使用的是 term frequency / inverse document frequency算法，**简称为 TF/IDF 算法**。

**Term frequency，TF词频**：搜索文本中的各个词条在 field 文本中出现了多少次，出现次数越多，就越相关。

![tfidf-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261417836.png)

举例：

* 有索引 index：medicine，搜索请求：阿莫西林
* doc1：阿莫西林胶囊是什么...阿莫西林胶囊能做什么...阿莫西林胶囊结构...
* doc2：本药店有阿莫西林胶囊、红霉素胶囊、青霉素胶囊...
* 结果：doc1 > doc2

**Inverse document frequency，IDF逆向文件频率**：搜索文本中的各个词条在整个索引的所有文档中出现了多少次，出现的次数越多，就越不相关.

![tfidf-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261417609.png)

![tfidf-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261417376.png)

举例：

* 有索引 index：medicine，搜索请求：阿莫西林胶囊 C市

* doc1 :  A市健康大药房简介，本药店有阿莫西林胶囊、红霉素胶囊、青霉素胶囊...

* doc2 :  B市民生大药房简介，本药店有阿莫西林胶囊、红霉素胶囊、青霉素胶囊...
* doc3：C市未来大药房简介，本药店有阿莫西林胶囊、红霉素胶囊、青霉素胶囊...

* 结论：整个索引库中出现频率少的词，相关度高，doc3 更相关

**Field-length norm**：field 长度，field 越长，相关度越弱

举例：

* 有索引 index：medicine，搜索请求：阿莫西林胶囊 A市

* doc1 : {"title":"A市健康大药房", "content ":"本药店红霉素胶囊、青霉素胶囊...(很多字，但没有阿莫西林胶囊出现)"}

* doc2 : {"title":"B市民生大药房简介", "content ":"本药店有阿莫西林胶囊、红霉素胶囊、青霉素胶囊...(很多字)"}
* 结论：doc1 > doc2

#### 16.1.2 _score计算

1. 对用户输入关键词分词，分成多个 term；
2. 每个分词分别计算每个匹配文档的 tf 和 idf 值；
3. 综合每个分词的 tf/idf 值，利用公式计算每个文档总分，倒序返回。

```json
GET /book/_search?explain=true
{
    "query": {
        "match": {
            "description": "java程序员"
        }
    }
}

// 返回
{
    "took" : 5,
    "timed_out" : false,
    "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits" : {
        "total" : {
            "value" : 2,
            "relation" : "eq"
        },
        "max_score" : 2.137549,
        "hits" : [
            {
                "_shard" : "[book][0]",
                "_node" : "MDA45-r6SUGJ0ZyqyhTINA",
                "_index" : "book",
                "_type" : "_doc",
                "_id" : "3",
                "_score" : 2.137549,
                "_source" : {
                    "name" : "spring开发基础",
                    "description" : "spring 在java领域非常流行，java程序员都在用。",
                    "studymodel" : "201001",
                    "price" : 88.6,
                    "timestamp" : "2019-08-24 19:11:35",
                    "pic" : "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
                    "tags" : [
                        "spring",
                        "java"
                    ]
                },
                "_explanation" : {
                    "value" : 2.137549,
                    "description" : "sum of:",
                    "details" : [
                        {
                            "value" : 0.7936629,
                            "description" : "weight(description:java in 0) [PerFieldSimilarity], result of:",
                            "details" : [
                                {
                                    "value" : 0.7936629,
                                    "description" : "score(freq=2.0), product of:",
                                    "details" : [
                                        {
                                            "value" : 2.2,
                                            "description" : "boost",
                                            "details" : [ ]
                                        },
                                        {
                                            "value" : 0.47000363,
                                            "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                                            "details" : [
                                                {
                                                    "value" : 2,
                                                    "description" : "n, number of documents containing term",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 3,
                                                    "description" : "N, total number of documents with field",
                                                    "details" : [ ]
                                                }
                                            ]
                                        },
                                        {
                                            "value" : 0.7675597,
                                            "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                                            "details" : [
                                                {
                                                    "value" : 2.0,
                                                    "description" : "freq, occurrences of term within document",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 1.2,
                                                    "description" : "k1, term saturation parameter",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 0.75,
                                                    "description" : "b, length normalization parameter",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 12.0,
                                                    "description" : "dl, length of field",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 35.333332,
                                                    "description" : "avgdl, average length of field",
                                                    "details" : [ ]
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "value" : 1.3438859,
                            "description" : "weight(description:程序员 in 0) [PerFieldSimilarity], result of:",
                            "details" : [
                                {
                                    "value" : 1.3438859,
                                    "description" : "score(freq=1.0), product of:",
                                    "details" : [
                                        {
                                            "value" : 2.2,
                                            "description" : "boost",
                                            "details" : [ ]
                                        },
                                        {
                                            "value" : 0.98082924,
                                            "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                                            "details" : [
                                                {
                                                    "value" : 1,
                                                    "description" : "n, number of documents containing term",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 3,
                                                    "description" : "N, total number of documents with field",
                                                    "details" : [ ]
                                                }
                                            ]
                                        },
                                        {
                                            "value" : 0.6227967,
                                            "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                                            "details" : [
                                                {
                                                    "value" : 1.0,
                                                    "description" : "freq, occurrences of term within document",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 1.2,
                                                    "description" : "k1, term saturation parameter",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 0.75,
                                                    "description" : "b, length normalization parameter",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 12.0,
                                                    "description" : "dl, length of field",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 35.333332,
                                                    "description" : "avgdl, average length of field",
                                                    "details" : [ ]
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            },
            {
                "_shard" : "[book][0]",
                "_node" : "MDA45-r6SUGJ0ZyqyhTINA",
                "_index" : "book",
                "_type" : "_doc",
                "_id" : "2",
                "_score" : 0.57961315,
                "_source" : {
                    "name" : "java编程思想",
                    "description" : "java语言是世界第一编程语言，在软件开发领域使用人数最多。",
                    "studymodel" : "201001",
                    "price" : 68.6,
                    "timestamp" : "2019-08-25 19:11:35",
                    "pic" : "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
                    "tags" : [
                        "java",
                        "dev"
                    ]
                },
                "_explanation" : {
                    "value" : 0.57961315,
                    "description" : "sum of:",
                    "details" : [
                        {
                            "value" : 0.57961315,
                            "description" : "weight(description:java in 0) [PerFieldSimilarity], result of:",
                            "details" : [
                                {
                                    "value" : 0.57961315,
                                    "description" : "score(freq=1.0), product of:",
                                    "details" : [
                                        {
                                            "value" : 2.2,
                                            "description" : "boost",
                                            "details" : [ ]
                                        },
                                        {
                                            "value" : 0.47000363,
                                            "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                                            "details" : [
                                                {
                                                    "value" : 2,
                                                    "description" : "n, number of documents containing term",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 3,
                                                    "description" : "N, total number of documents with field",
                                                    "details" : [ ]
                                                }
                                            ]
                                        },
                                        {
                                            "value" : 0.56055,
                                            "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                                            "details" : [
                                                {
                                                    "value" : 1.0,
                                                    "description" : "freq, occurrences of term within document",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 1.2,
                                                    "description" : "k1, term saturation parameter",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 0.75,
                                                    "description" : "b, length normalization parameter",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 19.0,
                                                    "description" : "dl, length of field",
                                                    "details" : [ ]
                                                },
                                                {
                                                    "value" : 35.333332,
                                                    "description" : "avgdl, average length of field",
                                                    "details" : [ ]
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            }
        ]
    }
}
```

### 16.2 doc value

**搜索的时候，要依靠倒排索引；排序的时候，需要依靠正排索引**，看到每个 document 的每个 field，然后进行排序，所谓的正排索引，其实就是 doc values

在建立索引的时候，一方面会建立倒排索引，以供搜索用；一方面会建立正排索引，也就是 doc values，以供排序，聚合，过滤等操作使用

doc values 是被保存在磁盘上的，此时如果内存足够，os 会自动将其缓存在内存中，性能还是会很高；如果内存不足够，os 会将其写入磁盘上

**倒排索引**

doc1：`{"content": "hello world you and me"}`

doc2：`{"content": "hi, world, how are you"}`

| term  | doc1 | doc2 |
| ----- | ---- | ---- |
| hello | √    |      |
| world | √    | √    |
| you   | √    | √    |
| and   | √    |      |
| me    | √    |      |
| hi    |      | √    |
| how   |      | √    |
| are   |      | √    |

搜索时：

* 搜索：`hello you` ，经过分词：hello, you

* hello → doc1

* you → doc1, doc2

当需要 sort by 时出现问题，当需要按照 content 进行排序时，由于不知道 content 的内容，无法排序

**正排索引**

doc1：`{ "name": "jack", "age": 27 }`

doc2：`{ "name": "tom", "age": 30 }`

| document | name | age  |
| -------- | ---- | ---- |
| doc1     | jack | 27   |
| doc2     | tom  | 30   |

### 16.3 query phase

**query phase 查询阶段：**

1. 搜索请求发送到某一个 coordinate node，构构建一个 priority queue，长度以 page 操作 from 和 size 为准，默认为 10

2. coordinate node 将请求转发到所有 shard，每个 shard 本地搜索，并构建一个本地的 priority queue

3. 各个 shard 将自己的 priority queue 返回给 coordinate node，并构建一个全局的 priority queue

> priority queue 中可以仅放 id 信息和 source 信息

**replica shard 如何提升搜索吞吐量**

一次请求要打到所有 shard 的一个 replica/primary 上去，如果每个 shard 都有多个 replica，那么同时并发过来的搜索请求可以同时打到其他的 replica 上去

### 16.4 fetch phase

fetch phase 拿取数据阶段的工作流程

1. coordinate node 构建完 priority queue 之后，就发送 mget 请求去所有 shard 上获取对应的 document

2. 各个 shard 将 document 返回给 coordinate node

3. coordinate node 将合并后的 document 结果返回给 client 客户端

一般搜索，如果不加 from 和 size，就默认搜索前 10 条，按照 _score 排序

## 16.5 搜索参数小总结

### 16.5.1 preference参数

**bouncing results 问题**，两个 document 排序，field 值相同；不同的 shard 上，可能排序不同；每次请求轮询打到不同的 replica shard 上；每次页面上看到的搜索结果的排序都不一样。这就是 bouncing result，也就是跳跃的结果。

搜索的时候，是轮询将搜索请求发送到每一个 replica shard（primary shard），但是在不同的 shard 上，可能 document 的排序不同

解决方案就是将 preference 设置为一个字符串，比如说 user_id，让每个 user 每次搜索的时候，都使用同一个 replica shard 去执行，就不会看到 bouncing results 了

preference 参数：决定了哪些 shard 会被用来执行搜索操作

* `_primary`：都从主分片上拿
* `_primary_first`：优先在主分片执行，如果主分片挂掉，会在副本执行请求；
* `_local`：搜索请求优先于在本地执行；
* `_only_node:xyz`：搜索请求只在在节点xyz执行；
* `_prefer_node:xyz`：搜索请求优先在节点xyz执行；
* `_shards:2,3`：搜索只在分片2、3执行

```
GET /_search?preference=_shards:2,3
```

### 16.5.2 timeout参数

已经讲解过原理了，主要就是限定在一定时间内，将部分获取到的数据直接返回，避免查询耗时过长

```json
GET /_search?timeout=10ms
```

### 16.5.3 routing参数

document 文档路由，_id路由，routing=user_id，这样的话可以让同一个 user 对应的数据到一个 shard 上去

```
GET /_search?routing=user123
```

### 16.5.4 search_type 参数

query_then_fetch：default 默认

dfs_query_then_fetch：可以提升revelance sort精准度
