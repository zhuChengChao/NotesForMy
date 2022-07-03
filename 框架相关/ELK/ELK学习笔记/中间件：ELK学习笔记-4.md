# 中间件：ELK学习笔记-4

> 很早之前粗略的学习过Lucene和Elasticsearch，然后最近实习过程中需要用到一些ES的基础操作，虽说使用起来没啥问题，但还是想稍微系统的学习一下，也能在秋招的时候稍微说道说道。
>
> 在 bilibili 上搜了一个课程，课程地址：[黑马：ELK高级搜索](https://www.bilibili.com/video/BV1Nt4y1m7qL)，跟着视频也做了些笔记，总的分了7篇，此为4/7篇。
>
> >学的很草率...空了再来完善:cry:

## 11. 索引Index入门

### 11.1 索引管理

直接 PUT 数据 `PUT index/_doc/1`，es 会自动生成索引，并建立动态映射 dynamic mapping。

在生产上，我们需要自己手动建立索引和映射，为了更好地管理索引。就像数据库的建表语句一样。

#### 11.1.1 创建索引

创建索引的语法：

```json
PUT /index
{
    "settings": { ... any settings, 如分片数副本数等 ... },
    "mappings": {  // 映射信息设置
        "properties" : {
            "field1" : { "type" : "text" }
        }
    },
    "aliases": {  // 别名
        "default_index": {}
    } 
}
```

举例：

```json
PUT /my_index
{
    "settings": {
        "number_of_shards": 1,    // 分片数
        "number_of_replicas": 1   // 副本数
    },
    "mappings": {
        "properties": {
            "field1":{
                "type": "text"
            },
            "field2":{
                "type": "text"
            }
        }
    },
    "aliases": {  // 设置别名
        "default_index": {}
    } 
}
```

插入数据：

```json
POST /my_index/_doc/1
{
	"field1":"java",
	"field2":"js"
}
```

查询数据（别名）都可以查到

`GET /my_index/_doc/1`

`GET /default_index/_doc/1`

#### 11.1.2 查询索引

查询索引：`GET /my_index/`

查询映射信息：`GET /my_index/_mapping`

查询设置信息：`GET /my_index/_setting`

#### 11.1.3 修改索引

修改副本数

```json
PUT /my_index/_settings
{
    "index" : {
        "number_of_replicas" : 2
    }
}
```

#### 11.1.4 删除索引

`DELETE /my_index`

`DELETE /索引名1,索引名2`

`DELETE /index_*`

`DELETE /_all`

为了安全起见，防止恶意删除索引，删除时必须指定索引名：

在 elasticsearch.yml 添加属性：`action.destructive_requires_name: true`

### 11.2 定制分词器

#### 11.2.1 默认的分词器

**分词三个组件：字符过滤器 character filter；分词器 tokenizer；词单元过滤器 token filter**。每个组件都可以自定义

* standard tokenizer：以单词边界进行切分

* standard token filter：什么都不做

* lowercase token filter：将所有字母转换为小写

* stop token filer（默认被禁用）：移除停用词，比如 a the it 等等

#### 11.2.2 修改分词器的设置

启用 english 停用词 token filter

```json
PUT /my_index
{
    "settings": {
        "analysis": {
            "analyzer": {   // 设置了一个分词器
                "es_std": {   
                    "type": "standard",
                    "stopwords": "_english_"
                }
            }
        }
    }
}
```

测试分词

```json
GET /my_index/_analyze
{
    "analyzer": "standard",   // 使用标准的分词器
    "text": "a dog is in the house"
}
// 分词结果是被分成了：dog,is,in,the,house

GET /my_index/_analyze
{
    "analyzer": "es_std",  // 使用上述设置的分词器，a/the/is/in 都没了
    "text":"a dog is in the house"
}
// 分词结果是被分成了：dog,house
```

#### 11.2.3 定制化自己的分词器

```json
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {  // 自定义预处理器组件
                "&_to_and": {
                    "type": "mapping",
                    "mappings": ["&=> and"]
                }
            },
            "filter": {  // 自定义filter组件
                "my_stopwords": {
                    "type": "stop",
                    "stopwords": ["the", "a"]
                }
            },
            "analyzer": {
                "my_analyzer": {  // 定义分词器，通过将三个组件进行结合
                    "type": "custom",  // 类型
                    "char_filter": ["html_strip", "&_to_and"],
                    "tokenizer": "standard",
                    "filter": ["lowercase", "my_stopwords"]
                }
            }
        }
    }
}
```

测试：

```json
GET /my_index/_analyze
{
    "analyzer": "my_analyzer",
    "text": "tom&jerry are a friend in the house, <a>, HAHA!!"
}
// 输出结果：tomandjerry,are,friend,in,house,haha
```

设置字段使用自定义分词器：

```json
PUT /my_index/_mapping/
{
    "properties": {
        "content": {
            "type": "text",
            "analyzer": "my_analyzer"
        }
    }
}
```

### 11.3 type底层结构及弃用原因

#### 11.3.1 type是什么

type，是一个 index 中用来区分类似数据的，类似的数据，但是可能有不同的 fields，而且有不同的属性来控制索引建立、分词器。

field 的 value，在底层的 lucene 中建立索引的时候，全部是 opaque bytes 类型，**不区分类型的**。

lucene 是没有 type 的概念的，在 document 中，**实际上将 type 作为一个 document 的 field 来存储**，即 `_type`，es 通过 `_type` 来进行 type 的过滤和筛选。

#### 11.3.2 ES中不同type存储机制

一个 index 中的多个 type，实际上是放在一起存储的，因此一个 index 下，不能有多个 type 重名而类型或者其他设置不同的，因为那样是无法处理的

```json
{
    "goods": {
        "mappings": {
            "electronic_goods": {  // 一个type
                "properties": {
                    "name": {
                        "type": "text"
                    },
                    "price": {
                        "type": "double"
                    },
                    "service_period": {
                        "type": "text"
                    }			
                }
            },
            "fresh_goods": {  // 一个type
                "properties": {
                    "name": {
                        "type": "text"
                    },
                    "price": {
                        "type": "double"
                    },
                    "eat_period": {
                        "type": "text"
                    }
                }
            }
        }
    }
}
```

```json
PUT /goods/electronic_goods/1
{
    "name": "小米空调",
    "price": 1999.0,
    "service_period": "one year"
}

PUT /goods/fresh_goods/1
{
    "name": "澳洲龙虾",
    "price": 199.0,
    "eat_period": "one week"
}
```

es 文档在底层的存储是这样子的

```json
{
    "goods": {
        "mappings": {
            "_type": {
                "type": "text",
                "index": "false"
            },
            "name": {
                "type": "text"
            },
            "price": {
                "type": "double"
            },
            "service_period": {
                "type": "string"
            },
            "eat_period": {
                "type": "string"
            }
        }
    }
}
```

底层数据存储格式：

```json
{
    "_type": "electronic_goods",  // 多了一个这个字段
    "name": "小米空调",
    "price": 1999.0,
    "service_period": "one year",
    "eat_period": ""
}

{
    "_type": "fresh_goods",  // 多了一个这个字段
    "name": "澳洲龙虾",
    "price": 199.0,
    "service_period": "",
    "eat_period": "one week"
}
```

#### 11.3.3 type弃用

同一索引下，不同 type 的数据存储其他 type 的 field 大量空值，造成资源浪费。

所以，不同类型数据，要放到不同的索引中。

> 在es9 中，将会彻底删除 type。

### 11.4 定制dynamic mapping

#### 11.4.1 定制dynamic mapping策略

**策略选择：**

* true：遇到陌生字段，就进行 dynamic mapping

* false：新检测到的字段将被忽略。这些字段将不会被索引，因此将无法搜索，但仍将出现在返回点击的源字段中，这些字段不会添加到映射中，必须显式添加新字段。

* strict：遇到陌生字段，就报错

创建 mapping：

```json
PUT /my_index
{
    "mappings": {
        "dynamic": "strict",  // true/false/strict
        "properties": {
            "title": {
                "type": "text"
            },
            "address": {
                "type": "object",
                "dynamic": "true"
            }
        }
    }
}
```

插入数据：strict的情况，会报错

```json
PUT /my_index/_doc/1
{
    "title": "my article",
    "content": "this is my article",
    "address": {
        "province": "guangdong",
        "city": "guangzhou"  // 新的字段field
    }
}

{
    "error": {
        "root_cause": [
            {
                "type": "strict_dynamic_mapping_exception",
                "reason": "mapping set to strict, dynamic introduction of [content] within [_doc] is not allowed"
            }
        ],
        "type": "strict_dynamic_mapping_exception",
        "reason": "mapping set to strict, dynamic introduction of [content] within [_doc] is not allowed"
    },
    "status": 400
}
```

插入数据：false 的情况

```json
// 可以正常插入，但是无法搜索 content 这个字段了
PUT /my_index/_doc/1
{
    "title": "my article",
    "content": "this is my article",
    "address": {
        "province": "guangdong",
        "city": "guangzhou"
    }
}

// 搜索content无匹配结果返回
GET /my_index/_search?q=content:"article"
// 搜索title有结果，且content会在source中
GET /my_index/_search?q=title:"article"
```

#### 11.4.2 自定义dynamic mapping策略

es 会根据传入的值，推断类型：

| JSON datatype         | Elasticsearch datatype                           |
| --------------------- | ------------------------------------------------ |
| null                  | No field is added                                |
| true / false          | boolean                                          |
| floating point number | float                                            |
| integer               | long                                             |
| object                | object                                           |
| array                 | Depends on the first non-null value in the array |
| string                | data/double/long/text/text(keyword)，会有探测器  |

**date_detection 日期探测**

默认会按照一定格式识别 date，比如 `yyyy-MM-dd`。但是如果某个 field 先过来一个 2017-01-01 的值，就会被自动 dynamic mapping 成 date，后面如果再来一个"hello world"之类的值，就会报错。可以手动关闭某个 type 的 date_detection，如果有需要，自己手动指定某个 field 为 date 类型。

```json
PUT /my_index
{
    "mappings": {
        "date_detection": false,  // 关闭date_detection 日期探测
        "properties": {
            "title": {
                "type": "text"
            },
            "address": {
                "type": "object",
                "dynamic": "true"
            }
        }
    }
}
```

测试

```json
PUT /my_index/_doc/1
{
    "title": "my article",
    "content": "this is my article",
    "address": {
        "province": "guangdong",
        "city": "guangzhou"
    },
    "post_date":"2019-09-10"
}
```

查看映射

```json
GET /my_index/_mapping

// 查看post_date类型
"post_date" : {
    "type" : "text",
    "fields" : {
        "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
        }
    }
},
```

**自定义日期格式**

```json
PUT my_index
{
    "mappings": {
        "dynamic_date_formats": ["MM/dd/yyyy"]
    }
}
```

插入数据

```json
PUT my_index/_doc/1
{
    "create_date": "09/25/2019"
}
```

查看映射

```json
GET /my_index/_mapping/
{
    "my_index" : {
        "mappings" : {
            "dynamic_date_formats" : [
                "MM/dd/yyyy"
            ],
            "properties" : {
                "create_date" : {
                    "type" : "date",
                    "format" : "MM/dd/yyyy"
                }
            }
        }
    }
}
```

**numeric_detection 数字探测**

虽然 json 支持本机浮点和整数数据类型，但某些应用程序或语言有时可能将数字呈现为字符串。通常正确的解决方案是显式地映射这些字段，但是可以启用数字检测（默认情况下禁用）来自动完成这些操作。

```json
// 创建索引，设置数字探测
PUT my_index
{
    "mappings": {
        "numeric_detection": true
    }
}

// 插入数据
PUT my_index/_doc/1
{
    "my_float": "1.0", 
    "my_integer": "1" 
}

// 查看映射
GET /my_index/_mapping/
{
    "my_index" : {
        "mappings" : {
            "numeric_detection" : true,
            "properties" : {
                "my_float" : {
                    "type" : "float"
                },
                "my_integer" : {
                    "type" : "long"
                }
            }
        }
    }
}
```

#### 11.4.3 定制dynamic mapping template

```json
PUT /my_index
{
    "mappings": {
        "dynamic_templates": [  // 定制模版，数组，可以定制多个
            { 
                "en": {
                    "match": "*_en",   // 以下划线en结尾
                    "match_mapping_type": "string",  // 是string类型
                    "mapping": {  // 做一下的映射
                        "type": "text",
                        "analyzer": "english"
                    }
                }                  
            }
        ]
    }
}
```

插入数据

```json
PUT /my_index/_doc/1
{
    "title": "this is my first article"
}

PUT /my_index/_doc/2
{
    "title_en": "this is my first article"
}
```

查看映射：

```json
GET my_index/_mapping

"properties" : {
    "title" : {
        "type" : "text",
        "fields" : {
            "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
            }
        }
    },
    "title_en" : {
        "type" : "text",
        "analyzer" : "english"
    }
}
```

搜索：

```json
GET my_index/_search?q=first  # 命中两条数据
GET my_index/_search?q=is     # 这里就只能命中一条数据了
```

* title 没有匹配到任何的 dynamic 模板，默认就是 standard 分词器，不会过滤停用词，is 会进入倒排索引，用 is 来搜索是可以搜索到的

* title_en 匹配到了dynamic 模板，就是 english 分词器，会过滤停用词，is 这种停用词就会被过滤掉，用 is 来搜索就搜索不到了

**模板写法**

```json
PUT my_index
{
    "mappings": {
        "dynamic_templates": [
            // 第一个模版
            {
                "integers": {
                    "match_mapping_type": "long",
                    "mapping": {
                        "type": "integer"
                    }
                }
            },
            // 第二个模版
            {
                "strings": {
                    "match_mapping_type": "string",
                    "mapping": {
                        "type": "text",
                        "fields": {
                            "raw": {
                                "type":  "keyword",
                                "ignore_above": 256
                            }
                        }
                    }
                }
            }
        ]
    }
}
```

模板参数：

```json
"match":   "long_*"
"unmatch": "*_text"
"match_mapping_type": "string"
"path_match":   "name.*"
"path_unmatch": "*.middle"
# 正则匹配
"match_pattern": "regex"
"match": "^profit_\d+$"
```

**场景**

1. 结构化搜索

   默认情况下，elasticsearch 将字符串字段映射为**带有子关键字字段的文本字段**。但是，如果只对结构化内容进行索引，而对全文搜索不感兴趣，则可以仅将“字段”映射为“关键字”。请注意，这意味着为了搜索这些字段，必须搜索索引所用的完全相同的值。

   ```json
   {
       "strings_as_keywords": {
           "match_mapping_type": "string",
           "mapping": {
               "type": "keyword"
           }
       }
   }
   ```

2. 仅搜索

   与前面的示例相反，如果您只关心字符串字段的全文搜索，并且不打算对字符串字段运行聚合、排序或精确搜索，您可以告诉弹性搜索将其仅映射为文本字段（这是5之前的默认行为）

   ```json
   {
       "strings_as_text": {
           "match_mapping_type": "string",
           "mapping": {
               "type": "text"
           }
       }
   }
   ```

3. norms 不关心评分

   norms 是指标时间的评分因素。如果您不关心评分，例如，如果您从不按评分对文档进行排序，则可以在索引中禁用这些评分因子的存储并节省一些空间。

   ```json
   {
       "strings_as_keywords": {
           "match_mapping_type": "string",
           "mapping": {
               "type": "text",
               "norms": false,  # 关闭
               "fields": {
                   "keyword": {
                       "type": "keyword",
                       "ignore_above": 256
                   }
               }
           }
       }
   }
   ```

### 11.5 零停机重建索引

**零停机重建索引** 

**场景**：一个 field 的设置是不能被修改的，如果要修改一个 field，那么应该重新按照新的 mapping，建立一个 index，然后将数据批量查询出来，重新用 bulk api 写入index 中。

批量查询的时候，建议采用 scroll api，并且采用多线程并发的方式来 reindex 数据，每次 scoll 就查询指定日期的一段数据，交给一个线程即可。

1. 一开始，依靠 dynamic mapping，插入数据，但是不小心有些数据是 2019-09-10 这种日期格式的，所以 title 这种 field 被自动映射为了 date 类型，实际上它应该是 string 类型的

   ```json
   PUT /my_index/_doc/1
   {
     "title": "2019-09-10"
   }
   
   PUT /my_index/_doc/2
   {
     "title": "2019-09-11"
   }
   ```

2. 当后期向索引中加入string类型的title值的时候，就会报错

   ```json
   PUT /my_index/_doc/3
   {
       "title": "my first article"
   }
   
   ```

// 插入后报错：
   {
    "error" : {
           "root_cause" : [
               {
                   "type" : "mapper_parsing_exception",
                   "reason" : "failed to parse field [title] of type [date] in document with id '3'. Preview of field's value: 'my first article'"
               }
           ],
           "type" : "mapper_parsing_exception",
           "reason" : "failed to parse field [title] of type [date] in document with id '3'. Preview of field's value: 'my first article'",
           "caused_by" : {
               "type" : "illegal_argument_exception",
               "reason" : "failed to parse date field [my first article] with format [strict_date_optional_time||epoch_millis]",
               "caused_by" : {
                   "type" : "date_time_parse_exception",
                   "reason" : "Failed to parse with all enclosed parsers"
               }
           }
       },
       "status" : 400
   }
   ```
   
3. 如果此时想修改title的类型，是不可能的

   ```json
   PUT /my_index/_mapping
   {
       "properties": {
           "title": {
               "type": "text"
           }
       }
   }
   
// 修改类型：报错
   {
    "error" : {
           "root_cause" : [
               {
                   "type" : "illegal_argument_exception",
                   "reason" : "mapper [title] of different type, current_type [date], merged_type [text]"
               }
           ],
           "type" : "illegal_argument_exception",
           "reason" : "mapper [title] of different type, current_type [date], merged_type [text]"
       },
       "status" : 400
   }
   
   ```

4. 此时，唯一的办法，就是进行 reindex，也就是说，重新建立一个索引，将旧索引的数据查询出来，再导入新索引。

5. 如果说旧索引的名字，是 old_index，新索引的名字是 new_index，终端 java 应用，已经在使用 old_index 在操作了，难道还要去停止 java 应用，修改使用的 index 为 new_index，才重新启动 java 应用吗？这个过程中，就会导致 java 应用停机，可用性降低。

6. 所以说，给 java 应用一个别名，这个别名是指向旧索引的，java 应用先用着，java 应用先用 prod_index alias 来操作，此时实际指向的是旧的 my_index

   ```json
   PUT /my_index/_alias/prod_index
   ```

7. 新建一个 index，调整其 title 的类型为 string

   ```json
   PUT /my_index_new
   {
       "mappings": {
           "properties": {
               "title": {
                   "type": "text"
               }
           }
       }
   }
   ```

8. 使用scroll api将数据批量查询出来

   ```json
   GET /my_index/_search?scroll=1m
   {
       "query": {
           "match_all": {}
       },
       "size": 20
   }
   
   ```

// 返回：
   {
    "_scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAADpAFjRvbnNUWVZaVGpHdklqOV9zcFd6MncAAAAAAAA6QRY0b25zVFlWWlRqR3ZJajlfc3BXejJ3AAAAAAAAOkIWNG9uc1RZVlpUakd2SWo5X3NwV3oydwAAAAAAADpDFjRvbnNUWVZaVGpHdklqOV9zcFd6MncAAAAAAAA6RBY0b25zVFlWWlRqR3ZJajlfc3BXejJ3",
       "took": 1,
       "timed_out": false,
       "_shards": {
           "total": 5,
           "successful": 5,
           "failed": 0
       },
       "hits": {
           "total": 3,
           "max_score": null,
           "hits": [
               {
                   "_index": "my_index",
                   "_type": "my_type",
                   "_id": "1",
                   "_score": null,
                   "_source": {
                       "title": "2019-01-02"
                   },
                   "sort": [
                       0
                   ]
               }
           ]
       }
   }
   ```
   
9. 采用 bulk api 将 scoll 查出来的一批数据，批量写入新索引

   ```json
   POST /_bulk
   { "index":  { "_index": "my_index_new", "_id": "1" }}
   { "title":    "2019-09-10" }
   ```

10. 反复循环 8~9，查询一批又一批的数据出来，采取 bulk api 将每一批数据批量写入新索引

11. 将 prod_index alias 切换到 my_index_new 上去，java 应用会直接通过 index 别名使用新的索引中的数据，java 应用程序不需要停机，零提交，高可用

    ```json
    POST /_aliases
    {
        "actions": [
            { "remove": { "index": "my_index", "alias": "prod_index" }},
            { "add":    { "index": "my_index_new", "alias": "prod_index" }}
        ]
    }
    ```

12. 直接通过prod_index别名来查询，是否ok

    ```json
    GET /prod_index/_search
    ```

**生产实践：基于 alias 对 client 透明切换 index**

```json
PUT /my_index_v1/_alias/my_index
```

client 对 my_index 进行操作；

reindex 操作，完成之后，切换 v1 到 v2

```json
POST /_aliases
{
    "actions": [
        { "remove": { "index": "my_index_v1", "alias": "my_index" }},
        { "add":    { "index": "my_index_v2", "alias": "my_index" }}
    ]
}
```

## 12. 中文分词器

### 12.1 Ik分词器安装使用

#### 12.1.1 中文分词器

standard 分词器，仅适用于英文。

```json
GET /_analyze
{
  "analyzer": "standard",
  "text": "中华人民共和国人民大会堂"
}
```

我们想要的效果是什么：中华人民共和国，人民大会堂

IK 分词器就是目前最流行的 es 中文分词器

#### 12.1.2 安装

官网：https://github.com/medcl/elasticsearch-analysis-ik

下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases

根据 es 版本下载相应版本包，然后解压到 `es/plugins/ik` 中，然后重启 es

#### 12.1.3 IK分词器基础知识

**ik_max_word**：会将文本做最细粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为“中华人民共和国，中华人民，中华，华人，人民共和国，人民大会堂，人民大会，大会堂”，会穷尽各种可能的组合；

**ik_smart**：会做最粗粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为“中华人民共和国，人民大会堂”。

```json
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": "中华人民共和国人民大会堂"
}

GET /_analyze
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国人民大会堂"
}
```

#### 12.1.4 ik分词器的使用

存储时，使用 ik_max_word，搜索时，使用 ik_smart

```json
PUT /my_index 
{
  "mappings": {
      "properties": {
        "text": {
          "type": "text",
          "analyzer": "ik_max_word",      // 存储时，使用 ik_max_word
          "search_analyzer": "ik_smart"   // 搜索时，使用 ik_smart
        }
      }
  }
}
```

搜索

```json
// 插入数据
PUT /my_index/_doc/1
{
  "text":"中华人民共和国人民大会堂"
}

// 搜索
GET /my_index/_search?q=中华人民共和国人民大会堂
GET /my_index/_search?q=会堂
```

### 12.2 IK配置文件

**IK配置文件**

IK 配置文件地址：`es/plugins/ik/config` 目录

* `IKAnalyzer.cfg.xml`：用来配置自定义词库

* `main.dic`：ik 原生内置的中文词库，总共有 27 万多条，只要是这些单词，都会被分在一起

* `preposition.dic`: 介词

* `quantifier.dic`：放了一些单位相关的词，量词

* `suffix.dic`：放了一些后缀

* `surname.dic`：中国的姓氏

* `stopword.dic`：英文停用词

IK 原生最重要的两个配置文件

* `main.dic`：包含了原生的中文词语，会按照这个里面的词语去分词

* `stopword.dic`：包含了英文的停用词

  停用词/stopword：a the and at but，一般，像停用词，会在分词的时候，直接被干掉，不会建立在倒排索引中


**自定义词库**

1. 自己建立词库：每年都会涌现一些特殊的流行词，网红，蓝瘦香菇，喊麦，鬼畜，一般不会在 ik 的原生词典里，自己补充自己的最新的词语，到 IK 的词库里面

   `IKAnalyzer.cfg.xml`：ext_dict，创建 `mydict.dic`

   补充自己的词语，然后需要重启 es，才能生效

2. 自己建立停用词库：比如了，的，啥，么，我们可能并不想去建立索引，让人家搜索

   `custom/ext_stopword.dic`，已经有了常用的中文停用词，可以补充自己的停用词，然后重启es

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict"></entry>  <!-- 创建好后讲创建的文件放入标签体内部 -->
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

### 12.3 使用MySQL热更新词库

#### 12.3.1 热更新

每次都是在 es 的扩展词典中，手动添加新词语，很坑

1. 每次添加完，都要重启 es 才能生效，非常麻烦

2. es 是分布式的，可能有数百个节点，你不能每次都一个一个节点上面去修改

es 不停机，直接我们在外部某个地方添加新的词语，es中立即热加载到这些新词语

热更新的方案

1. 方式1：基于 Ik 分词器原生支持的热更新方案，部署一个 web 服务器，提供一个 http 接口，通过 modified 和 tag 两个 http 响应头，来提供词语的热更新

2. 方式2：修改 IK 分词器源码，然后手动支持从 MySQL 中每隔一定时间，自动加载新的词库

用第二种方案，第一种，IK Git 社区官方都不建议采用，觉得不太稳定

####  12.3.2 修改源码步骤

1. 下载源码：https://github.com/medcl/elasticsearch-analysis-ik/releases

   IK 分词器，是个标准的 java maven 工程，直接导入 IDEA 就可以看到源码

2. 修改源

   `org.wltea.analyzer.dic.Dictionary` 类，160行 Dictionary 单例类的初始化方法，在这里需要创建一个我们自定义的线程，并且启动它

   `org.wltea.analyzer.dic.HotDictReloadThread` 类：就是死循环，不断调用`Dictionary.getSingleton().reLoadMainDict()`，去重新加载词典

   Dictionary 类，399行：`this.loadMySQLExtDict();` 加载 mysql 字典。

   Dictionary 类，609行：`this.loadMySQLStopwordDict();` 加载 mysql 停用词

   config 下 创建 jdbc-reload.properties。mysql 的配置文件

   ```properties
   jdbc.url=jdbc:mysql://localhost:3306/es?serverTimezone=GMT
   jdbc.user=root
   jdbc.password=root
   # 建立了一个热词表
   jdbc.reload.sql=select word from hot_words
   # 建立一个停用词表
   jdbc.reload.stopword.sql=select stopword as word from hot_stopwords  
   jdbc.reload.interval=5000
   ```

3. mvn package 打包代码：`target\releases\elasticsearch-analysis-ik-7.3.0.zip`，将打成的 jar 包替换原先的 ik 分词的 jar 包

4. 解压缩 ik 压缩包

   将 mysql 驱动 jar 包，放入 es 的 lib 目录下

5. 修改 jdbc 相关配置，即将上述的 jdbc-reload.properties 放入到 ik分词器插件的 config 目录中

6. 重启 es：观察日志，日志中就会显示我们打印的那些东西，比如加载了什么配置，加载了什么词语，什么停用词

7. 在 mysql 中的相应表中添加词库与停用词

8. 分词实验，验证热更新生效

## 13. Java API 实现索引管理

```java
@SpringBootTest(classes = SearchApplication.class)
@RunWith(SpringRunner.class)
public class TestIndex {

    @Autowired
    RestHighLevelClient client;

}
```

### 13.1 创建索引

```java
/**
 * 测试同步方式创建索引
 */
@Test
public void testCreateIndex() throws IOException {

    // 1.创建索引请求
    CreateIndexRequest createIndexRequest = new CreateIndexRequest("my_index");
    // 2. 设置参数
    createIndexRequest.settings(Settings.builder().put("number_of_shards", "1").put("number_of_replicas", "0"));

    // -----------指定映射 的几种方式 start ----------- //
    // 方式1：指定映射
    createIndexRequest.mapping(" {\n" +
                               " \"properties\": {\n" +
                               "   \"name\":{\n" +
                               "     \"type\":\"keyword\"\n" +
                               "   },\n" +
                               "   \"description\": {\n" +
                               "      \"type\": \"text\"\n" +
                               "   },\n" +
                               "   \"price\":{\n" +
                               "     \"type\":\"long\"\n" +
                               "   },\n" +
                               "   \"pic\":{\n" +
                               "     \"type\":\"text\",\n" +
                               "     \"index\":false\n" +
                               "   }\n" +
                               " }\n" +
                               "}", XContentType.JSON);

    // 方式2：指定映射
    /*Map<String, Object> name = new HashMap<>();
        name.put("type", "keyword");
        Map<String, Object> description = new HashMap<>();
        description.put("type", "text");
        Map<String, Object> price = new HashMap<>();
        price.put("type", "long");
        Map<String, Object> pic = new HashMap<>();
        pic.put("type", "text");
        pic.put("index", "false");
        // 构建properties
        Map<String, Object> properties = new HashMap<>();
        properties.put("name", name);
        properties.put("description", description);
        properties.put("price", price);
        properties.put("pic", pic);
        // 构建mapping
        Map<String, Object> mapping = new HashMap<>();
        properties.put("properties", properties);
        // 加入请求中
        createIndexRequest.mapping(mapping);*/

    // 方式3：指定映射
    /*XContentBuilder builder = XContentFactory.jsonBuilder();
        builder.startObject();
        {
            builder.startObject("properties");
            {
                builder.startObject("message");
                {
                    builder.field("type", "text");
                }
                builder.endObject();
            }
            builder.endObject();
        }
        builder.endObject();
        createIndexRequest.mapping(builder);*/
    // -----------指定映射 的几种方式 end ----------- //

    // 4.其他设置
    // 设置别名
    createIndexRequest.alias(new Alias("index_new"));
    // 额外参数
    // 设置超时时间
    createIndexRequest.setTimeout(TimeValue.timeValueMinutes(2));
    // 设置主节点超时时间
    createIndexRequest.setMasterTimeout(TimeValue.timeValueMinutes(1));
    // 在创建索引API返回响应之前等待的活动分片副本的数量，以int形式表示
    createIndexRequest.waitForActiveShards(ActiveShardCount.from(2));
    createIndexRequest.waitForActiveShards(ActiveShardCount.DEFAULT);

    // 3.执行：操作索引的客户端
    IndicesClient indices = client.indices();
    // 执行创建索引库
    CreateIndexResponse createIndexResponse = indices.create(createIndexRequest, RequestOptions.DEFAULT);

    // 4. 得到响应（全部）
    boolean acknowledged = createIndexResponse.isAcknowledged();
    // 得到响应 指示是否在超时前为索引中的每个分片启动了所需数量的碎片副本
    boolean shardsAcknowledged = createIndexResponse.isShardsAcknowledged();

    System.out.println("相应结果：" + acknowledged);
    System.out.println(shardsAcknowledged);
}

/**
 * 测试异步方式创建索引
 */
@Test
public void testSyncCreateIndex() throws IOException {

    // 1.创建索引请求
    CreateIndexRequest createIndexRequest = new CreateIndexRequest("my_index");
    // 2. 设置参数
    createIndexRequest.settings(Settings.builder().put("number_of_shards", "1").put("number_of_replicas", "0"));

    // 3.指定映射 TODO：好像没起作用
    Map<String, Object> name = new HashMap<>();
    name.put("type", "keyword");
    Map<String, Object> description = new HashMap<>();
    description.put("type", "text");
    Map<String, Object> price = new HashMap<>();
    price.put("type", "long");
    Map<String, Object> pic = new HashMap<>();
    pic.put("type", "text");
    pic.put("index", "false");
    // 构建properties
    Map<String, Object> properties = new HashMap<>();
    properties.put("name", name);
    properties.put("description", description);
    properties.put("price", price);
    properties.put("pic", pic);
    // 构建mapping
    Map<String, Object> mapping = new HashMap<>();
    properties.put("properties", properties);
    // 加入请求中
    createIndexRequest.mapping(mapping);

    // 4.其他设置
    // 设置别名
    createIndexRequest.alias(new Alias("index_new"));
    // 额外参数
    // 设置超时时间
    createIndexRequest.setTimeout(TimeValue.timeValueMinutes(2));
    // 设置主节点超时时间
    createIndexRequest.setMasterTimeout(TimeValue.timeValueMinutes(1));
    // 在创建索引API返回响应之前等待的活动分片副本的数量，以int形式表示
    createIndexRequest.waitForActiveShards(ActiveShardCount.from(2));
    createIndexRequest.waitForActiveShards(ActiveShardCount.DEFAULT);

    // 4.执行：操作索引的客户端
    // 监听方法
    ActionListener<CreateIndexResponse> listener =
        new ActionListener<CreateIndexResponse>() {

        @Override
        public void onResponse(CreateIndexResponse createIndexResponse) {
            // 4. 得到响应（全部）
            boolean acknowledged = createIndexResponse.isAcknowledged();
            // 得到响应 指示是否在超时前为索引中的每个分片启动了所需数量的碎片副本
            boolean shardsAcknowledged = createIndexResponse.isShardsAcknowledged();

            System.out.println("创建索引成功：" + acknowledged);
            System.out.println(shardsAcknowledged);
        }

        @Override
        public void onFailure(Exception e) {
            System.out.println("创建索引失败");
            e.printStackTrace();
        }
    };

    // 异步：执行创建索引库
    IndicesClient indices = client.indices();
    indices.createAsync(createIndexRequest, RequestOptions.DEFAULT, listener);

    // 避免直接关闭看不到结果，这里延时一下
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

}
```

### 13.2 删除索引

```java
/**
 * 删除索引
 * @throws IOException
 */
@Test
public void testDeleteIndex() throws IOException {

    // 删除索引请求对象
    DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("my_index");
    // 操作索引的客户端
    IndicesClient indices = client.indices();
    // 执行删除索引
    AcknowledgedResponse delete = indices.delete(deleteIndexRequest, RequestOptions.DEFAULT);
    // 得到响应
    boolean acknowledged = delete.isAcknowledged();
    System.out.println(acknowledged);
}


/**
     * 异步删除索引库
     * @throws IOException
     */
@Test
public void testDeleteIndexAsync() throws IOException {
    // 删除索引对象
    DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("my_index");
    // 操作索引的客户端
    IndicesClient indices = client.indices();

    // 监听方法
    ActionListener<AcknowledgedResponse> listener =
        new ActionListener<AcknowledgedResponse>() {
        @Override
        public void onResponse(AcknowledgedResponse deleteIndexResponse) {
            System.out.println("删除索引成功");
            System.out.println(deleteIndexResponse.toString());
        }

        @Override
        public void onFailure(Exception e) {
            System.out.println("删除索引失败");
            e.printStackTrace();
        }
    };
    // 执行删除索引
    indices.deleteAsync(deleteIndexRequest, RequestOptions.DEFAULT, listener);

    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

### 12.3 判断索引是否存在

```java
/**
 * 判断索引是否存在
 */
@Test
public void  testExistIndex() throws IOException {

    GetIndexRequest request = new GetIndexRequest("my_index");
    // 从主节点返回本地信息或检索状态
    request.local(false);
    // 以适合人类的格式返回结果
    request.humanReadable(true);
    // 是否返回每个索引的所有默认设置
    request.includeDefaults(false);

    boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
    System.out.println(exists);
}
```

### 12.4 开关索引

```java
/**
 * 开启索引
 * @throws IOException
 */
@Test
public void testOpenIndex() throws IOException {

    OpenIndexRequest request = new OpenIndexRequest("book");

    OpenIndexResponse openIndexResponse = client.indices().open(request, RequestOptions.DEFAULT);
    boolean acknowledged = openIndexResponse.isAcknowledged();
    System.out.println("开启索引结果；"+acknowledged);
}

/**
 * 关闭索引
 * @throws IOException
 */
@Test
public void testCloseIndex() throws IOException {
    CloseIndexRequest request = new CloseIndexRequest("book");
    AcknowledgedResponse closeIndexResponse = client.indices().close(request, RequestOptions.DEFAULT);
    boolean acknowledged = closeIndexResponse.isAcknowledged();
    System.out.println("关闭结果结果："+acknowledged);

}
```

