# 中间件：ELK学习笔记-6

> 很早之前粗略的学习过Lucene和Elasticsearch，然后最近实习过程中需要用到一些ES的基础操作，虽说使用起来没啥问题，但还是想稍微系统的学习一下，也能在秋招的时候稍微说道说道。
>
> 在 bilibili 上搜了一个课程，课程地址：[黑马：ELK高级搜索](https://www.bilibili.com/video/BV1Nt4y1m7qL)，跟着视频也做了些笔记，总的分了7篇，此为6/7篇。
>
> > 学的很草率...空了再来完善:cry:

## 17. 聚合入门

### 17.1 聚合示例

#### 需求：计算每个 studymodel 下的商品数量

sql语句： `select studymodel, count(*) from book group by studymodel`

```json
GET /book/_search
{
    "size": 0, 
    "query": {
        "match_all": {}
    }, 
    "aggs": {
        "group_by_model": {
            "terms": { "field": "studymodel" }
        }
    }
}
```

结果：

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
            "value" : 3,
            "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
    },
    "aggregations" : {
        "group_by_model" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
                {
                    "key" : "201001",
                    "doc_count" : 2
                },
                {
                    "key" : "201002",
                    "doc_count" : 1
                }
            ]
        }
    }
}
```

#### 需求：计算每个 tags 下的商品数量

设置字段 `"fielddata": true`

> 因为之前的 tags的字段
>
> ```json
> "tags" : 
> {
>       "type" : "text",
>       "fields" : {
>            "keyword" : {
>                "type" : "keyword",
>                "ignore_above" : 256
>            }
>       }
> },
> ```
>
> 直接用下面的查询会报错：`"reason" : "Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [tags] in order to load field data by uninverting the inverted index. Note that this can use significant memory."`

```json
PUT /book/_mapping/
{
    "properties": {
        "tags": {
            "type": "text",
            "fielddata": true
        }
    }
}
```

查询

```json
GET /book/_search
{
    "size": 0, 
    "query": {
        "match_all": {}
    }, 
    "aggs": {
        "group_by_tags": {
            "terms": { "field": "tags" }
        }
    }
}
```

结果：

```json
"aggregations" : {
    "group_by_tags" : {
        "doc_count_error_upper_bound" : 0,
        "sum_other_doc_count" : 0,
        "buckets" : [
            {
                "key" : "dev",
                "doc_count" : 2
            },
            {
                "key" : "java",
                "doc_count" : 2
            },
            {
                "key" : "bootstrap",
                "doc_count" : 1
            },
            {
                "key" : "spring",
                "doc_count" : 1
            }
        ]
    }
```

#### 需求：加上搜索条件后计算每个tags下的商品数量

```json
GET /book/_search
{
    "size": 0, 
    "query": {
        "match": {
            "description": "java程序员"
        }
    }, 
    "aggs": {
        "group_by_tags": {
            "terms": { "field": "tags" }
        }
    }
}
```

#### 需求：先按照tag分组 再算每组的平均值

```json
GET /book/_search
{
    "size": 0,
    "aggs" : {
        "group_by_tags" : {
            "terms" : { 
                "field" : "tags" 
            },
            "aggs" : {
                "avg_price" : {
                    "avg" : { "field" : "price" }
                }
            }
        }
    }
}
```

#### 需求：计算每个tag下的商品的平均价格 且按照平均价格降序排序

```json
GET /book/_search
{
    "size": 0,
    "aggs" : {
        "group_by_tags" : {
            "terms" : { 
                "field" : "tags",
                "order": {
                    "avg_price": "desc"  # 调用了子聚合中的结果
                }
            },
            "aggs" : {
                "avg_price" : {
                    "avg" : { "field" : "price" }
                }
            }
        }
    }
}
```

#### 需求：按照指定的价格范围区间进行分组 然后在每组内再按照 tag 进行分组 最后再计算每组的平均价格

```json
GET /book/_search
{
    "size": 0,
    "aggs": {
        "group_by_price": {
            "range": {
                "field": "price",
                "ranges": [
                    {
                        "from": 0,
                        "to": 40
                    },
                    {
                        "from": 40,
                        "to": 60
                    },
                    {
                        "from": 60,
                        "to": 80
                    }
                ]
            },
            "aggs": {
                "group_by_tags": {
                    "terms": {
                        "field": "tags"
                    },
                    "aggs": {
                        "average_price": {
                            "avg": {
                                "field": "price"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

### 17.2 两个核心概念：bucket和metric

### 17.2.1 bucket：一个数据分组

| city | name   |
| ---- | ------ |
| 北京 | 张三   |
| 北京 | 李四   |
| 天津 | 王五   |
| 天津 | 赵六   |
| 天津 | 王麻子 |

按照城市分组，就是分桶。`select city, count(*) from table group by city`

划分出来两个bucket，一个是北京 bucket，一个是天津 bucket

北京 bucket：包含了2个人，张三，李四

天津 bucket：包含了3个人，王五，赵六，王麻子

### 17.2.2 metric：对一个数据分组执行的统计

metric，就是对一个 bucket 执行的某种聚合分析的操作，比如说求 平均值，求最大值，求最小值

`select studymodel, count(*) from book group studymodel`

bucket：group by studymodel → 那些 studymodel 相同的数据，就会被划分到一个 bucket 中

metric：`count(*)`，对每个 user_id bucket 中所有的数据，计算一个数量。还有 `avg()`，`sum()`，`max()`，`min()`

### 17.3 电视案例 

#### 创建索引及映射

```json
PUT /tvs
PUT /tvs/_mapping
{			
    "properties": {
        "price": {  // 价格
            "type": "long"
        },
        "color": {  // 颜色
            "type": "keyword"
        },
        "brand": {  // 品牌
            "type": "keyword"
        },
        "sold_date": {  // 售卖日期
            "type": "date"
        }
    }
}
```

#### 插入数据

```json
POST /tvs/_bulk
{ "index": {}}
{ "price" : 1000, "color" : "红色", "brand" : "长虹", "sold_date" : "2019-10-28" }
{ "index": {}}
{ "price" : 2000, "color" : "红色", "brand" : "长虹", "sold_date" : "2019-11-05" }
{ "index": {}}
{ "price" : 3000, "color" : "绿色", "brand" : "小米", "sold_date" : "2019-05-18" }
{ "index": {}}
{ "price" : 1500, "color" : "蓝色", "brand" : "TCL", "sold_date" : "2019-07-02" }
{ "index": {}}
{ "price" : 1200, "color" : "绿色", "brand" : "TCL", "sold_date" : "2019-08-19" }
{ "index": {}}
{ "price" : 2000, "color" : "红色", "brand" : "长虹", "sold_date" : "2019-11-05" }
{ "index": {}}
{ "price" : 8000, "color" : "红色", "brand" : "三星", "sold_date" : "2020-01-01" }
{ "index": {}}
{ "price" : 2500, "color" : "蓝色", "brand" : "小米", "sold_date" : "2020-02-12" }
```

#### 需求1：统计哪种颜色的电视销量最高

```json
GET /tvs/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color"
            }
        }
    }
}
```

查询条件解析

* size：**只获取聚合结果**，而不要执行聚合的原始数据
* aggs：固定语法，要对一份数据执行分组聚合操作
  * popular_colors：就是对每个aggs，都要起一个名字，
    * terms：根据字段的值进行分组
      * field：根据指定的字段的值进行分组

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
            "value" : 8,
            "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
    },
    "aggregations" : {
        "popular_colors" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
                {
                    "key" : "红色",
                    "doc_count" : 4
                },
                {
                    "key" : "绿色",
                    "doc_count" : 2
                },
                {
                    "key" : "蓝色",
                    "doc_count" : 2
                }
            ]
        }
    }
}
```

返回结果解析：

* hits.hits：我们指定了size是0，所以hits.hits就是空的
* aggregations：聚合结果
  * popular_color：我们指定的某个聚合的名称
    * buckets：根据我们指定的 field 划分出的 buckets
      * key：每个bucket对应的那个值
      * doc_count：这个bucket分组内，有多少个数据，数量，其实就是这种颜色的销量

每种颜色对应的 bucket 中的数据的默认的排序规则：按照 doc_count 降序排序

#### 需求2：统计每种颜色电视平均价格

```json
GET /tvs/_search
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": { 
            "avg_price": { 
               "avg": {
                  "field": "price" 
               }
            }
         }
      }
   }
}
```

在一个 aggs 执行的 bucket 操作（terms），平级的 json 结构下，再加一个 aggs，这个第二个 aggs 内部，同样取个名字，执行一个 metric 操作，avg，对之前的每个 bucket 中的数据的指定的 field，price field，求一个平均值

返回：

```json
{
    "took" : 4,
    "timed_out" : false,
    "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits" : {
        "total" : {
            "value" : 8,
            "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
    },
    "aggregations" : {
        "colors" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
                {
                    "key" : "红色",
                    "doc_count" : 4,
                    "avg_price" : {
                        "value" : 3250.0
                    }
                },
                {
                    "key" : "绿色",
                    "doc_count" : 2,
                    "avg_price" : {
                        "value" : 2100.0
                    }
                },
                {
                    "key" : "蓝色",
                    "doc_count" : 2,
                    "avg_price" : {
                        "value" : 2000.0
                    }
                }
            ]
        }
    }
}
```

buckets，除了 key 和 doc_count

* avg_price：我们自己取的 metric aggs 的名字
  * value：我们的metric计算的结果，每个bucket中的数据的price字段求平均值后的结果

相当于 sql：`select avg(price) from tvs group by color`

#### 需求3：继续下钻分析

每个颜色下，平均价格，和每个颜色下，每个品牌的平均价格

```json
GET /tvs/_search 
{
    "size": 0,
    "aggs": {
        "group_by_color": {
            "terms": {
                "field": "color"  // 每个颜色下
            },
            "aggs": {
                "color_avg_price": {
                    "avg": {
                        "field": "price"  // 每个颜色下的平均价格
                    }
                },
                "group_by_brand": {
                    "terms": {
                        "field": "brand"  // 每个颜色下的每个品牌
                    },
                    "aggs": {
                        "brand_avg_price": {
                            "avg": {
                                "field": "price"  // 每个颜色下的每个品牌价格
                            }
                        }
                    }
                }
            }
        }
    }
}
```

#### 需求4：更多的 metric

count：在 bucket 中按照 terms 计数，自动就会有一个 doc_count，就相当于是 count

avg：es 中也存在 avg 函数，求平均值

max：求一个 bucket 内，指定 field 值最大的那个数据

min：求一个 bucket 内，指定 field 值最小的那个数据

sum：求一个 bucket 内，指定 field 值的总和

需求：求出每个颜色的销量、平均价格、最大价格、最小价格、价格总和

```json
GET /tvs/_search
{
    "size" : 0,
    "aggs": {
        "colors": {
            "terms": {
                "field": "color"
            },
            "aggs": {
                "avg_price": { "avg": { "field": "price" } },
                "min_price" : { "min": { "field": "price"} }, 
                "max_price" : { "max": { "field": "price"} },
                "sum_price" : { "sum": { "field": "price" } } 
            }
        }
    }
}
```

结果：

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
            "value" : 8,
            "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
    },
    "aggregations" : {
        "colors" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
                {
                    "key" : "红色",
                    "doc_count" : 4,
                    "max_price" : {
                        "value" : 8000.0
                    },
                    "min_price" : {
                        "value" : 1000.0
                    },
                    "avg_price" : {
                        "value" : 3250.0
                    },
                    "sum_price" : {
                        "value" : 13000.0
                    }
                },
                {
                    "key" : "绿色",
                    "doc_count" : 2,
                    "max_price" : {
                        "value" : 3000.0
                    },
                    "min_price" : {
                        "value" : 1200.0
                    },
                    "avg_price" : {
                        "value" : 2100.0
                    },
                    "sum_price" : {
                        "value" : 4200.0
                    }
                },
                {
                    "key" : "蓝色",
                    "doc_count" : 2,
                    "max_price" : {
                        "value" : 2500.0
                    },
                    "min_price" : {
                        "value" : 1500.0
                    },
                    "avg_price" : {
                        "value" : 2000.0
                    },
                    "sum_price" : {
                        "value" : 4000.0
                    }
                }
            ]
        }
    }
}

```

#### 需求5：划分范围 histogram 以价格为区间

**划分范围 histogram，求出价格每 2000 为一个区间，每个区间的销售总额**

```json
GET /tvs/_search
{
    "size" : 0,
    "aggs":{
        "price_histogram":{
            "histogram":{ 
                "field": "price",
                "interval": 2000
            },
            "aggs":{
                "income": {
                    "sum": { 
                        "field" : "price"
                    }
                }
            }
        }
    }
}
```

histogram：类似于 terms，也是进行 bucket 分组操作，接收一个 field，按照这个 field 的值的各个范围区间，进行 bucket 分组操作

```json
"histogram":{ 
    "field": "price",
    "interval": 2000
}
```

interval：2000，划分范围，0~2000，2000~4000，4000~6000，6000~8000，8000~10000，buckets

bucket 有了之后，一样的，去对每个 bucket 执行 avg，count，sum，max，min，等各种 metric 操作，聚合分析

#### 需求6：按照日期分组聚合 求每个月销售数

date_histogram，按照我们指定的某个 date 类型的日期 field，以及日期 interval，按照一定的日期间隔，去划分bucket

min_doc_count：即使某个日期 interval，2017-01-01~2017-01-31 中，一条数据都没有，那么这个区间也是要返回的，不然默认是会过滤掉这个区间的

extended_bounds：min，max，划分bucket的时候，会限定在这个起始日期，和截止日期内

```json
GET /tvs/_search
{
    "size" : 0,
    "aggs": {
        "data_sales": {
            "date_histogram": {
                "field": "sold_date",
                "interval": "month", 
                "format": "yyyy-MM-dd",
                "min_doc_count" : 0, 
                "extended_bounds" : { 
                    "min" : "2019-01-01",
                    "max" : "2020-12-31"
                }
            }
        }
    }
}
```

#### 需求7：统计每季度每个品牌的销售额 及每个季度销售总额

```json
GET /tvs/_search 
{
    "size": 0,
    "aggs": {
        "group_by_sold_date": {
            "date_histogram": {
                "field": "sold_date",
                "interval": "quarter",
                "format": "yyyy-MM-dd",
                "min_doc_count": 0,
                "extended_bounds": {
                    "min": "2019-01-01",
                    "max": "2020-12-31"
                }
            },
            "aggs": {
                "group_by_brand": {
                    "terms": {
                        "field": "brand"
                    },
                    "aggs": {
                        "sum_price": {
                            "sum": {
                                "field": "price"
                            }
                        }
                    }
                },
                "total_sum_price": {
                    "sum": {
                        "field": "price"
                    }
                }
            }
        }
    }
}
```

#### 需求8：搜索与聚合结合 查询某个品牌按颜色销量

搜索与聚合可以结合起来。

sql：`select count(*) from tvs where brand like "%小米%" group by color`

es 中 aggregation，scope，任何的聚合，都必须在搜索出来的结果数据中进行，搜索结果，就是聚合分析操作的scope

```json
GET /tvs/_search 
{
    "size": 0,
    "query": {
        "term": {
            "brand": {
                "value": "小米"
            }
        }
    },
    "aggs": {
        "group_by_color": {
            "terms": {
                "field": "color"
            }
        }
    }
}
```

#### 需求9：global bucket 单个品牌与所有品牌销量对比

aggregation，scope，一个聚合操作，必须在 query 的搜索结果范围内执行

出来两个结果，一个结果，是基于 query 搜索结果来聚合的；一个结果，是对所有数据执行聚合的

```json
GET /tvs/_search 
{
    "size": 0, 
    "query": {
        "term": {
            "brand": {
                "value": "小米"
            }
        }
    },
    "aggs": {
        "single_brand_avg_price": {
            "avg": {
                "field": "price"
            }
        },
        "all": {
            "global": {},  // 加了 global 后就是对所有数据执行聚合的，超出了上述的查询范围
            "aggs": {
                "all_brand_avg_price": {
                    "avg": {
                        "field": "price"
                    }
                }
            }
        }
    }
}
```

#### 需求10：过滤+聚合 统计价格大于1200的电视平均价格

搜索+过滤+聚合

```json
GET /tvs/_search 
{
    "size": 0,
    "query": {
        "constant_score": {
            "filter": {
                "range": {
                    "price": {
                        "gte": 1200
                    }
                }
            }
        }
    },
    "aggs": {
        "avg_price": {
            "avg": {
                "field": "price"
            }
        }
    }
}
```

#### 需求11：bucket filter 统计品牌最近一个月的平均价格

在 bucket 中添加过滤器，**bucket filter：对不同的 bucket 下的 aggs，进行 filter**

```json
GET /tvs/_search 
{
    "size": 0,
    "query": {  // 1.先查询出 "小米" 品牌的数据
        "term": {
            "brand": {
                "value": "小米"
            }
        }
    },
    "aggs": {
        "recent_150d": {   // 2.聚合1
            "filter": {  // 2.1 在聚合前 先添加一个过滤操作
                "range": {
                    "sold_date": {
                        "gte": "now-150d"  // 最近150天
                    }
                }
            },
            "aggs": {  // 2.2 按照平均价格聚合
                "recent_150d_avg_price": {
                    "avg": {
                        "field": "price"
                    }
                }
            }
        },
        "recent_140d": {  // 2.聚合2
            "filter": {  // 2.1 在聚合前 先添加一个过滤操作
                "range": {
                    "sold_date": {
                        "gte": "now-140d"
                    }
                }
            },
            "aggs": {  // 2.2 按照平均价格聚合
                "recent_140d_avg_price": {
                    "avg": {
                        "field": "price"
                    }
                }
            }
        },
        "recent_130d": {  // 2.聚合2
            "filter": {  // 2.1 在聚合前 先添加一个过滤操作
                "range": {
                    "sold_date": {
                        "gte": "now-130d"
                    }
                }
            },
            "aggs": {  // 2.2 按照平均价格聚合
                "recent_130d_avg_price": {
                    "avg": {
                        "field": "price"
                    }
                }
            }
        }
    }
}
```

`aggs.filter`：针对的是**聚合去做的**

如果放 query 里面的 filter，**是全局的**，会对所有的数据都有影响

但是，如果，比如说，你要统计，长虹电视，最近1个月的平均值；最近3个月的平均值；最近6个月的平均值，这时候就需要使用 bucket filter 了

#### 需求12：按每种颜色的平均销售额降序排序

```json
GET /tvs/_search 
{
    "size": 0,
    "aggs": {
        "group_by_color": {
            "terms": {
                "field": "color",
                "order": {
                    "avg_price": "asc"
                }
            },
            "aggs": {
                "avg_price": {
                    "avg": {
                        "field": "price"
                    }
                }
            }
        }
    }
}
```

父层聚合能直接使用子聚合的结果进行排序，相当于 sql 子表数据字段可以立刻使用

#### 需求13：按每种颜色的每种品牌平均销售额降序排序

```json
GET /tvs/_search  
{
    "size": 0,
    "aggs": {
        "group_by_color": {
            "terms": {
                "field": "color"
            },
            "aggs": {
                "group_by_brand": {
                    "terms": {
                        "field": "brand",
                        "order": {
                            "avg_price": "desc"
                        }
                    },
                    "aggs": {
                        "avg_price": {
                            "avg": {
                                "field": "price"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

## 18. Java API 实现聚合

简单聚合，多种聚合，详见代码

```java
package cn.xyc.test;

import cn.xyc.es.SearchApplication;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.aggregations.Aggregation;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.aggregations.Aggregations;
import org.elasticsearch.search.aggregations.bucket.histogram.*;
import org.elasticsearch.search.aggregations.bucket.terms.Terms;
import org.elasticsearch.search.aggregations.bucket.terms.TermsAggregationBuilder;
import org.elasticsearch.search.aggregations.metrics.*;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.IOException;
import java.util.List;

@SpringBootTest(classes = SearchApplication.class)
@RunWith(SpringRunner.class)
public class TestAggs {

    @Autowired
    RestHighLevelClient client;

    /**
     * 需求一：按照颜色分组，计算每个颜色卖出的个数
     * @throws IOException
     */
    @Test
    public void testAggs() throws IOException {

        // 对应的 Rest 查询
        // GET /tvs/_search
        // {
        //     "size": 0,
        //     "query": {"match_all": {}},
        //     "aggs": {
        //       "group_by_color": {
        //         "terms": {
        //             "field": "color"
        //         }
        //     }
        // }
        // }

        //1 构建请求
        SearchRequest searchRequest = new SearchRequest("tvs");
        // 封装请求体
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        searchSourceBuilder.size(0);
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());
        // 构建聚合
        TermsAggregationBuilder termsAggregationBuilder = AggregationBuilders.terms("group_by_color").field("color");
        searchSourceBuilder.aggregation(termsAggregationBuilder);
        // 请求体放入请求头
        searchRequest.source(searchSourceBuilder);

        // 2 执行
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        // 3 获取结果
        //   Rest 查询的结果：
        //   "aggregations" : {
        //       "group_by_color" : {
        //           "doc_count_error_upper_bound" : 0,
        //           "sum_other_doc_count" : 0,
        //            "buckets" : [
        //           {
        //               "key" : "红色",
        //               "doc_count" : 4
        //           },
        //           {
        //               "key" : "绿色",
        //                   "doc_count" : 2
        //           },
        //           {
        //               "key" : "蓝色",
        //                   "doc_count" : 2
        //           }
        //           ]
        //       }
        Aggregations aggregations = searchResponse.getAggregations();
        Terms group_by_color = aggregations.get("group_by_color");
        List<? extends Terms.Bucket> buckets = group_by_color.getBuckets();
        for (Terms.Bucket bucket : buckets) {
            String key = bucket.getKeyAsString();
            System.out.println("key:"+key);

            long docCount = bucket.getDocCount();
            System.out.println("docCount:"+docCount);

            System.out.println("=================================");
        }
    }

    /**
     * 需求二：按照颜色分组，计算每个颜色卖出的个数，每个颜色卖出的平均价格
     * @throws IOException
     */
    @Test
    public void testAggsAndAvg() throws IOException {

        // 对应的 Rest 查询
        // GET /tvs/_search
        // {
        //     "size": 0,
        //     "query": {"match_all": {}},
        //     "aggs": {
        //     "group_by_color": {
        //         "terms": {
        //             "field": "color"
        //         },
        //         "aggs": {
        //             "avg_price": {
        //                 "avg": {
        //                     "field": "price"
        //                 }
        //             }
        //         }
        //     }
        // }

        // 1 构建请求
        SearchRequest searchRequest = new SearchRequest("tvs");

        // 请求体
        SearchSourceBuilder searchSourceBuilder=new SearchSourceBuilder();
        searchSourceBuilder.size(0);
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());
        // 聚合请求构造
        TermsAggregationBuilder termsAggregationBuilder = AggregationBuilders.terms("group_by_color").field("color");
        // terms聚合下填充一个子聚合
        AvgAggregationBuilder avgAggregationBuilder = AggregationBuilders.avg("avg_price").field("price");
        termsAggregationBuilder.subAggregation(avgAggregationBuilder);
        // 聚合内容翻入请求中
        searchSourceBuilder.aggregation(termsAggregationBuilder);
        //请求体放入请求头
        searchRequest.source(searchSourceBuilder);

        // 2 执行
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        // 3.获取结果
        // {
        //     "key" : "红色",
        //      "doc_count" : 4,
        //      "avg_price" : {
        //        "value" : 3250.0
        //       }
        // }
        Aggregations aggregations = searchResponse.getAggregations();
        Terms group_by_color = aggregations.get("group_by_color");
        List<? extends Terms.Bucket> buckets = group_by_color.getBuckets();
        for (Terms.Bucket bucket : buckets) {
            String key = bucket.getKeyAsString();
            System.out.println("key:"+key);

            long docCount = bucket.getDocCount();
            System.out.println("docCount:"+docCount);

            Avg avg_price = bucket.getAggregations().get("avg_price");
            double value = avg_price.getValue();
            System.out.println("value:"+value);

            System.out.println("=================================");
        }
    }

    /**
     * 需求三：按照颜色分组，计算每个颜色卖出的个数，以及每个颜色卖出的平均值、最大值、最小值、总和。
     * @throws IOException
     */
    @Test
    public void testAggsAndMore() throws IOException {

        // 对应的 Rest 查询
        // GET /tvs/_search
        // {
        //     "size" : 0,
        //     "aggs": {
        //      "group_by_color": {
        //         "terms": {
        //             "field": "color"
        //         },
        //         "aggs": {
        //             "avg_price": { "avg": { "field": "price" } },
        //             "min_price" : { "min": { "field": "price"} },
        //             "max_price" : { "max": { "field": "price"} },
        //             "sum_price" : { "sum": { "field": "price" } }
        //         }
        //     }
        // }

        // 1.构建请求
        SearchRequest searchRequest = new SearchRequest("tvs");

        // 请求体
        SearchSourceBuilder searchSourceBuilder=new SearchSourceBuilder();
        searchSourceBuilder.size(0);
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());

        // 聚合参数构造
        TermsAggregationBuilder termsAggregationBuilder = AggregationBuilders.terms("group_by_color").field("color");
        // termsAggregationBuilder里放入多个子聚合
        AvgAggregationBuilder avgAggregationBuilder = AggregationBuilders.avg("avg_price").field("price");
        MinAggregationBuilder minAggregationBuilder = AggregationBuilders.min("min_price").field("price");
        MaxAggregationBuilder maxAggregationBuilder = AggregationBuilders.max("max_price").field("price");
        SumAggregationBuilder sumAggregationBuilder = AggregationBuilders.sum("sum_price").field("price");

        termsAggregationBuilder.subAggregation(avgAggregationBuilder);
        termsAggregationBuilder.subAggregation(minAggregationBuilder);
        termsAggregationBuilder.subAggregation(maxAggregationBuilder);
        termsAggregationBuilder.subAggregation(sumAggregationBuilder);
        // 聚合内容放入请求体
        searchSourceBuilder.aggregation(termsAggregationBuilder);
        //请求体放入请求头
        searchRequest.source(searchSourceBuilder);

        //2 执行
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        //3 获取结果
        // {
        //     "key" : "红色",
        //     "doc_count" : 4,
        //     "max_price" : {
        //          "value" : 8000.0
        //     },
        //     "min_price" : {
        //          "value" : 1000.0
        // },
        //     "avg_price" : {
        //         "value" : 3250.0
        // },
        //     "sum_price" : {
        //         "value" : 13000.0
        // }
        Aggregations aggregations = searchResponse.getAggregations();
        Terms group_by_color = aggregations.get("group_by_color");
        List<? extends Terms.Bucket> buckets = group_by_color.getBuckets();
        for (Terms.Bucket bucket : buckets) {
            String key = bucket.getKeyAsString();
            System.out.println("key:"+key);

            long docCount = bucket.getDocCount();
            System.out.println("docCount:"+docCount);

            Aggregations aggregations1 = bucket.getAggregations();

            Max max_price = aggregations1.get("max_price");
            double maxPriceValue = max_price.getValue();
            System.out.println("maxPriceValue:"+maxPriceValue);

            Min min_price = aggregations1.get("min_price");
            double minPriceValue = min_price.getValue();
            System.out.println("minPriceValue:"+minPriceValue);

            Avg avg_price = aggregations1.get("avg_price");
            double avgPriceValue = avg_price.getValue();
            System.out.println("avgPriceValue:"+avgPriceValue);

            Sum sum_price = aggregations1.get("sum_price");
            double sumPriceValue = sum_price.getValue();
            System.out.println("sumPriceValue:"+sumPriceValue);

            System.out.println("=================================");
        }
    }

    /**
     * 需求四：按照售价每2000价格划分范围，算出每个区间的销售总额 histogram
     * @throws IOException
     */
    @Test
    public void testAggsAndHistogram() throws IOException {

        // 对应的 Rest 查询
        // GET /tvs/_search
        // {
        //     "size" : 0,
        //     "aggs":{
        //      "by_histogram":{
        //         "histogram":{
        //             "field": "price",
        //             "interval": 2000
        //         },
        //         "aggs":{
        //             "income": {
        //                 "sum": {
        //                     "field" : "price"
        //                 }
        //             }
        //         }
        //     }
        // }

        // 1.构建请求
        SearchRequest searchRequest=new SearchRequest("tvs");

        // 构建请求体
        SearchSourceBuilder searchSourceBuilder=new SearchSourceBuilder();
        // 请求体中需要查询的内容
        searchSourceBuilder.size(0);
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());
        // 聚合参数构建
        HistogramAggregationBuilder histogramAggregationBuilder = AggregationBuilders.histogram("by_histogram").field("price").interval(2000);
        // 子聚合内容
        SumAggregationBuilder sumAggregationBuilder = AggregationBuilders.sum("income").field("price");
        histogramAggregationBuilder.subAggregation(sumAggregationBuilder);
        // 聚合内容放入请求体中
        searchSourceBuilder.aggregation(histogramAggregationBuilder);
        // 请求体放入请求头
        searchRequest.source(searchSourceBuilder);

        // 2 执行
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        // 3 获取结果
        // {
        //     "key" : 0.0,
        //     "doc_count" : 3,
        //      income" : {
        //          "value" : 3700.0
        //       }
        // }
        Aggregations aggregations = searchResponse.getAggregations();
        Histogram group_by_color = aggregations.get("by_histogram");
        List<? extends Histogram.Bucket> buckets = group_by_color.getBuckets();
        for (Histogram.Bucket bucket : buckets) {
            String keyAsString = bucket.getKeyAsString();
            System.out.println("keyAsString:"+keyAsString);
            long docCount = bucket.getDocCount();
            System.out.println("docCount:"+docCount);

            Aggregations aggregations1 = bucket.getAggregations();
            Sum income = aggregations1.get("income");
            double value = income.getValue();
            System.out.println("value:"+value);

            System.out.println("=================================");
        }
    }

    /**
     * 需求五：计算每个季度的销售总额
     * @throws IOException
     */
    @Test
    public void testAggsAndDateHistogram() throws IOException {
        // GET /tvs/_search
        // {
        //     "size" : 0,
        //     "aggs": {
        //     "sales": {
        //         "date_histogram": {
        //             "field": "sold_date",
        //             "interval": "quarter",
        //             "format": "yyyy-MM-dd",
        //             "min_doc_count" : 0,
        //             "extended_bounds" : {
        //                 "min" : "2019-01-01",
        //                 "max" : "2020-12-31"
        //             }
        //         },
        //         "aggs": {
        //             "income": {
        //                 "sum": {
        //                     "field": "price"
        //                 }
        //             }
        //         }
        //     }
        // }

        // 1.构建请求
        SearchRequest searchRequest = new SearchRequest("tvs");

        // 请求体构建
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        // 查询请求放入请求体中
        searchSourceBuilder.size(0);
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());
        // 构建聚合内容
        DateHistogramAggregationBuilder dateHistogramAggregationBuilder = AggregationBuilders
                .dateHistogram("date_histogram")
                .field("sold_date")
                .calendarInterval(DateHistogramInterval.QUARTER)
                .format("yyyy-MM-dd")
                .minDocCount(0)
                .extendedBounds(new ExtendedBounds("2019-01-01", "2020-12-31"));
        SumAggregationBuilder sumAggregationBuilder = AggregationBuilders.sum("income").field("price");
        dateHistogramAggregationBuilder.subAggregation(sumAggregationBuilder);
        // 聚合请求放入请求体中
        searchSourceBuilder.aggregation(dateHistogramAggregationBuilder);
        // 请求体放入请求头
        searchRequest.source(searchSourceBuilder);

        //2 执行
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        //3 获取结果
        // {
        //     "key_as_string" : "2019-01-01",
        //      "key" : 1546300800000,
        //      "doc_count" : 0,
        //      "income" : {
        //         "value" : 0.0
        //      }
        // }
        Aggregations aggregations = searchResponse.getAggregations();
        ParsedDateHistogram date_histogram = aggregations.get("date_histogram");
        List<? extends Histogram.Bucket> buckets = date_histogram.getBuckets();
        for (Histogram.Bucket bucket : buckets) {
            String keyAsString = bucket.getKeyAsString();
            System.out.println("keyAsString:"+keyAsString);
            long docCount = bucket.getDocCount();
            System.out.println("docCount:"+docCount);

            Aggregations aggregations1 = bucket.getAggregations();
            Sum income = aggregations1.get("income");
            double value = income.getValue();
            System.out.println("value:"+value);
            System.out.println("====================");
        }
    }

}
```

## 19. ES7：SQL新特性

### 19.1 快速入门

**需求：查询所有**

```json
POST /_sql?format=txt
{
    "query": "SELECT * FROM tvs"
}

# 结果
     brand   |     color     |     price     |       sold_date        
------------ +---------------+---------------+------------------------
长虹           |红色             |1000           |2019-10-28T00:00:00.000Z
长虹           |红色             |2000           |2019-11-05T00:00:00.000Z
小米           |绿色             |3000           |2019-05-18T00:00:00.000Z
TCL           |蓝色             |1500           |2019-07-02T00:00:00.000Z
TCL           |绿色             |1200           |2019-08-19T00:00:00.000Z
长虹           |红色             |2000           |2019-11-05T00:00:00.000Z
三星           |红色             |8000           |2020-01-01T00:00:00.000Z
小米           |蓝色             |2500           |2020-02-12T00:00:00.000Z
```

**需求：求出每个颜色的销量、平均价格、最大价格、最小价格、价格总和**

```json
POST /_sql?format=txt
{
  "query": "select color, avg(price), min(price), max(price), sum(price) from tvs group by color"
}

# 结果
     color     |  avg(price)   |  min(price)   |  max(price)   |  sum(price)   
---------------+---------------+---------------+---------------+---------------
红色             |3250.0         |1000.0         |8000.0         |13000.0        
绿色             |2100.0         |1200.0         |3000.0         |4200.0         
蓝色             |2000.0         |1500.0         |2500.0         |4000.0      
```

### 19.2 启动方式

1. http 请求（19.1 快速入门）

2. 客户端：其中 `elasticsearch-sql-cli.bat`

   ```bash
   sql> select * from tvs;
        brand     |     color     |     price     |       sold_date
   ---------------+---------------+---------------+------------------------
   长虹             |红色             |1000           |2019-10-28T00:00:00.000Z
   长虹             |红色             |2000           |2019-11-05T00:00:00.000Z
   小米             |绿色             |3000           |2019-05-18T00:00:00.000Z
   TCL            |蓝色             |1500           |2019-07-02T00:00:00.000Z
   TCL            |绿色             |1200           |2019-08-19T00:00:00.000Z
   长虹             |红色             |2000           |2019-11-05T00:00:00.000Z
   三星             |红色             |8000           |2020-01-01T00:00:00.000Z
   小米             |蓝色             |2500           |2020-02-12T00:00:00.000Z
   
   sql> show tables;
                name             |     type      |     kind
   ------------------------------+---------------+---------------
   .apm-agent-configuration      |TABLE          |INDEX
   .apm-custom-link              |TABLE          |INDEX
   .kibana                       |VIEW           |ALIAS
   .kibana-event-log-7.8.0       |VIEW           |ALIAS
   .kibana-event-log-7.8.0-000001|TABLE          |INDEX
   .kibana_1                     |TABLE          |INDEX
   .kibana_task_manager          |VIEW           |ALIAS
   .kibana_task_manager_1        |TABLE          |INDEX
   book                          |TABLE          |INDEX
   company                       |TABLE          |INDEX
   tvs                           |TABLE          |INDEX
   website                       |TABLE          |INDEX
   ```

3. 代码（见19.6 Java代码实现SQL功能）

### 19.3 显示方式

![显示方式](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261417934.png)

```json
// txt 方式
POST /_sql?format=txt
{
  "query": "select color, avg(price), min(price), max(price), sum(price) from tvs group by color"
}
     color     |  avg(price)   |  min(price)   |  max(price)   |  sum(price)   
---------------+---------------+---------------+---------------+---------------
红色             |3250.0         |1000.0         |8000.0         |13000.0        
绿色             |2100.0         |1200.0         |3000.0         |4200.0         
蓝色             |2000.0         |1500.0         |2500.0         |4000.0        

// csv方式：
POST /_sql?format=csv
{
  "query": "select color, avg(price), min(price), max(price), sum(price) from tvs group by color"
}
color,avg(price),min(price),max(price),sum(price)
红色,3250.0,1000.0,8000.0,13000.0
绿色,2100.0,1200.0,3000.0,4200.0
蓝色,2000.0,1500.0,2500.0,4000.0

// tsv方式：POST /_sql?format=tsv
color	avg(price)	min(price)	max(price)	sum(price)
红色	3250.0	1000.0	8000.0	13000.0
绿色	2100.0	1200.0	3000.0	4200.0
蓝色	2000.0	1500.0	2500.0	4000.0
```

### 19.4 SQL翻译

```json
POST /_sql/translate
{
    "query": "SELECT * FROM tvs "
}

// 返回：
{
    "size" : 1000,
    "_source" : false,
    "stored_fields" : "_none_",
    "docvalue_fields" : [
        {
            "field" : "brand"
        },
        {
            "field" : "color"
        },
        {
            "field" : "price"
        },
        {
            "field" : "sold_date",
            "format" : "epoch_millis"
        }
    ],
    "sort" : [
        {
            "_doc" : {
                "order" : "asc"
            }
        }
    ]
}
```

### 19.5 与其他DSL结合

```json
POST /_sql?format=txt
{
    "query": "SELECT * FROM tvs",
    "filter": {
        "range": {
            "price": {
                "gte" : 1200,
                "lte" : 2000
            }
        }
    }
}

# 结果：
     brand     |     color     |     price     |       sold_date        
---------------+---------------+---------------+------------------------
长虹             |红色             |2000           |2019-11-05T00:00:00.000Z
TCL            |蓝色             |1500           |2019-07-02T00:00:00.000Z
TCL            |绿色             |1200           |2019-08-19T00:00:00.000Z
长虹             |红色             |2000           |2019-11-05T00:00:00.000Z
```

### 19.6 Java代码实现sql功能

1. 前提 es 拥有白金版功能：kibana中管理 → 许可管理 → 开启白金版试用

2. 导入依赖

   ```xml
   <dependency>
       <groupId>org.elasticsearch.plugin</groupId>
       <artifactId>x-pack-sql-jdbc</artifactId>
       <version>7.3.0</version>
   </dependency>
   
   <repositories>
       <repository>
           <id>elastic.co</id>
           <url>https://artifacts.elastic.co/maven</url>
       </repository>
   </repositories>
   ```

3. 代码

   ```java
   public static void main(String[] args) {
       try  {
           // 创建连接
           Connection connection = DriverManager.getConnection("jdbc:es://http://localhost:9200");
           // 创建statement
           Statement statement = connection.createStatement();
           // 执行sql语句，获取结果
           ResultSet results = statement.executeQuery(
               "select * from tvs"
           );
           while(results.next()){
               System.out.println(results.getString(1));
               System.out.println(results.getString(2));
               System.out.println(results.getString(3));
               System.out.println(results.getString(4));
               System.out.println("============================");
           }
       }catch (Exception e){
           e.printStackTrace();
       }
   }
   ```

