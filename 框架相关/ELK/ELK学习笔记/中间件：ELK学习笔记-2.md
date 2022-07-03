# 中间件：ELK学习笔记-2

> 很早之前粗略的学习过Lucene和Elasticsearch，然后最近实习过程中需要用到一些ES的基础操作，虽说使用起来没啥问题，但还是想稍微系统的学习一下，也能在秋招的时候稍微说道说道。
>
> 在 bilibili 上搜了一个课程，课程地址：[黑马：ELK高级搜索](https://www.bilibili.com/video/BV1Nt4y1m7qL)，跟着视频也做了些笔记，总的分了7篇，此为2/7篇。
>
> > 学的很草率...空了再来完善:cry:

## 5. ES 快速入门

### 5.1 document的数据格式

1. 应用系统的数据结构都是面向对象的，具有复杂的数据结构

2. 对象存储到数据库，需要将关联的复杂对象属性插到另一张表，查询时再拼接起来。

3. es 面向文档，文档中存储的数据结构，与对象一致。所以**一个对象可以直接存成一个文档**。

4. es 的 document 用 json 数据格式来表达。

例如：班级和学生关系

```java
public class Student {
    private String id;
    private String name;

    private String classInfoId;  
}

private class ClassInfo {
    private String id;
    private String className;
}
```

数据库中要设计所谓的一对多，多对一的两张表，外键等。查询出来时，还要关联，MyBatis 写映射文件，很繁琐。

而在 es 中，一个学生存成文档如下：

```json
{
    "id":"1",
    "name": "张三",
    "last_name": "zhang",
    "classInfo": {
        "id": "1",
        "className": "三年二班",     
    }
}
```

### 5.2 图书网站商品管理案例：背景介绍

有一个售卖图书的网站，需要为其基于 ES 构建一个后台系统，提供以下功能：

1. 对商品信息进行CRUD（增删改查）操作

2. 执行简单的结构化查询
3. 可以执行简单的全文检索，以及复杂的phrase（短语）检索
4. 对于全文检索的结果，可以进行高亮显示
5. 对数据进行简单的聚合分析

### 5.3 简单的集群管理

#### 5.3.1 快速检查集群的健康状况

es 提供了一套 API，叫做 cat api，可以查看 es 中各种各样的数据，通过 Kibana 的 Dev Tools 输入：`GET /_cat/health?v`

![CATAPI使用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415990.png)

快速了解集群的健康状况：green、yellow、red

* green：每个索引的 primary shard 和 replica shard 都是 active 状态的
* yellow：每个索引的 primary shard 都是 active 状态的，但是部分 replica shard 不是 active 状态，处于不可用的状态
* red：不是所有索引的 primary shard 都是 active 状态的，部分索引有数据丢失了

#### 5.3.2 快速查看集群中有哪些索引

`GET /_cat/indices?v`

![快速查看集群中有哪些索引](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415354.png)

#### 5.3.3 简单的索引操作

创建索引：`PUT /demo_index?pretty`

> 在任意的查询字符串中增加`pretty`参数，会让 es **美化输出(pretty-print)** json 响应以便更加容易阅读。

```json
// 返回：
{
    "acknowledged" : true,
    "shards_acknowledged" : true,
    "index" : "demo_index"
}
```

删除索引：`DELETE /demo_index?pretty`

```json
// 返回：
{
  "acknowledged" : true
}
```

### 5.4 商品document的CRUD

#### 5.4.1 新建图书索引

首先建立图书索引：book

语法：`put /索引名`，即：`PUT /book`

![创建book索引](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415941.png)

#### 5.4.2 新增图书：新增文档

语法：`PUT /index/_doc/id`

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

#### 5.4.3 查询图书：检索文档

语法：`GET /index/type/id`

查看图书：`GET /book/_doc/1`  就可看到 json 形式的文档，方便程序解析。

```json
GET /book/_doc/1

// 返回：
{
    "_index" : "book",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "found" : true,
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
}
```

> 为方便查看索引中的数据，Kibana 可以如下操作：Kibana → Discover → Create index pattern 中的Index pattern填刚刚创建的book
>
> ![Kibana查看索引-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415270.png)
>
> ![Kibana查看索引-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415326.png)
>
> 选择 Kibana的discover 就可看到数据：
>
> ![Kibana查看索引-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415948.png)
>
> 点击 json 还可以看到原始数据
>
> ![Kibana查看索引-4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415763.png)

#### 5.4.4 修改图书：替换操作

替换操作是整体覆盖，要带上所有信息。

```json
PUT /book/_doc/1
{
    "name": "Bootstrap开发教程1",
    "description": "Bootstrap是由Twitter推出的一个前台页面开发css框架，是一个非常流行的开发框架，此框架集成了多种页面效果。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长css页面开发的程序人员）轻松的实现一个css，不受浏览器限制的精美界面css效果。",
    "studymodel": "201002",
    "price":38.6,
    "timestamp":"2019-08-25 19:11:35",
    "pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg",
    "tags": [ "bootstrap", "开发"]
}
```

#### 5.4.5 修改图书：更新文档

语法：`POST /{index}/type/{id}/_update` 或者 `POST /{index}/_update/{id}`

```json
POST /book/_update/1/ 
{
    "doc": {
        "name": " Bootstrap开发教程高级"
    }
}

// 返回：
{
    "_index" : "book",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 3,
    "result" : "updated",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 4,
    "_primary_term" : 1
}
```

#### 5.4.6 删除图书：删除文档

```json
DELETE /book/_doc/1

// 返回
{
    "_index" : "book",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 4,
    "result" : "deleted",  # 删除操作
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 5,
    "_primary_term" : 1
}
```

## 6. 文档document入门

### 6.1 默认自带字段解析

```json
GET /book/_doc/2

// 返回：
{
    "_index" : "book",
    "_type" : "_doc",
    "_id" : "2",
    "_version" : 1,
    "_seq_no" : 1,
    "_primary_term" : 1,
    "found" : true,
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
```

#### 6.1.1 _index

-  含义：此文档属于哪个索引
-  原则：类似数据放在一个索引中。数据库中表的定义规则。如图书信息放在 book 索引中，员工信息放在 employee 索引中。各个索引存储和搜索时互不影响。
-  定义规则：英文小写。尽量不要使用特殊字符。

#### 6.1.2 _type

-  含义：类别。
-  注意：以后的 ES9 将彻底删除此字段，所以当前版本在不断弱化 type。不需要关注。见到 _type 都为 _doc。

#### 6.1.3 _id

* 含义：文档的唯一标识。就像表的id主键。结合索引可以标识和定义一个文档。
* 生成：手动（`put /index/_doc/id`）、自动

**创建索引时，不同数据放到不同索引中**

### 6.2 生成文档id

#### 6.2.1 手动生成 id

场景：数据从其他系统导入时，本身有唯一主键。如数据库中的图书、员工信息等。

用法：`put /index/_doc/id`

```json
PUT /test_index/_doc/1
{
    "test_field": "test"
}

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "result" : "created",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 0,
    "_primary_term" : 1
}
```

#### 6.2.2 自动生成 id

用法：`POST /index/_doc`

```json
POST /test_index/_doc
{
    "test_field": "test1"
}

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "P4K3aX0BuTXKjrjqjq6B",
    "_version" : 1,
    "result" : "created",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 1,
    "_primary_term" : 1
}
```

自动 id 特点：长度为 20 个字符，URL 安全，base64 编码，GUID，分布式生成不冲突 

### 6.3 _source字段

####  6.3.1 _source

* 含义：插入数据时的所有字段和值。当get获取数据时，在_source字段中原样返回。
* 语法：`GET /book/_doc/1`

```json
{
    "_index" : "book",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "found" : true,
    "_source" : {  # source字段内容
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
}
```

#### 6.3.2 定制返回字段

就像 SQL 不要 `select *`，而要 `select name,price from book …` 一样，查询时也可以指定字段返回：

```json
GET /book/_doc/1?_source_includes=name,price

// 返回：
{
    "_index" : "book",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 6,
    "_primary_term" : 1,
    "found" : true,
    "_source" : {
        "price" : 38.6,
        "name" : "Bootstrap开发"
    }
}
```

### 6.4 文档的替换与删除

#### 6.4.1 全量替换

执行两次，返回结果中版本号（_version）在不断上升。此过程为全量替换。

```json
// 第一次执行
PUT /test_index/_doc/1
{
    "test_field": "test"
}

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 2,
    "result" : "updated",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 2,
    "_primary_term" : 1
}

// 第二次执行：全量替换
PUT /test_index/_doc/1
{
    "test_field": "test"
}

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 2,   # version+1了
    "result" : "updated",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 12,
    "_primary_term" : 1
}
```

**实质**：旧文档的内容不会立即删除，只是标记为 deleted。适当的时机，集群会将这些文档删除。

#### 6.4.2 强制创建

为防止覆盖原有数据，我们在新增时，设置为**强制创建**，不会覆盖原有文档。

语法：`PUT /index/_doc/id/_create`

```json
PUT /test_index/_doc/1/_create
{
    "test_field": "test"
}

// 返回：
{
    "error" : {
        "root_cause" : [
            {
                "type" : "version_conflict_engine_exception",
                "reason" : "[1]: version conflict, document already exists (current version [2])",
                "index_uuid" : "JmfPLjAZSeOGl0xcCPGjXg",
                "shard" : "0",
                "index" : "test_index"
            }
        ],
        "type" : "version_conflict_engine_exception",
        "reason" : "[1]: version conflict, document already exists (current version [2])",
        "index_uuid" : "JmfPLjAZSeOGl0xcCPGjXg",
        "shard" : "0",
        "index" : "test_index"
    },
    "status" : 409
}
```

#### 6.4.3 删除

```json
DELETE /test_index/_doc/1

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 3,
    "result" : "deleted",  # 标记删除
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 3,
    "_primary_term" : 1
}
```

**实质**：旧文档的内容不会立即删除（lazy delete），只是标记为 deleted。适当的时机，集群会将这些文档删除。

### 6.5 局部替换

使用 `PUT /index/type/id` 为文档全量替换，需要将文档所有数据提交。

partial update / 局部替换则只修改变动字段。

**用法**：

```json
post /index/type/id/_update 
{
    "doc": {
        "field"："value"
    }
}
```

**内部原理**：内部与全量替换是一样的，旧文档标记为删除，新建一个文档。

**优点**：

- 大大减少网络传输次数和流量，提升性能
- 减少并发冲突发生的概率。

**演示**：

* 插入文档

```json
PUT /test_index/_doc/5
{
    "test_field1": "itcast",
    "test_field2": "itheima"
}

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "5",
    "_version" : 1,
    "result" : "created",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 4,
    "_primary_term" : 1
}
```

* 修改字段

```json
POST /test_index/_doc/5/_update
{
    "doc": {
        "test_field2": " itheima 2"
    }
}

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "5",
    "_version" : 2,
    "result" : "updated",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 5,
    "_primary_term" : 1
}
```

### 6.6 使用脚本更新

es 可以内置脚本执行复杂操作。例如 painless 脚本。

> 注意：groovy 脚本在 es6 以后就不支持了。原因是耗内存，不安全远程注入漏洞。

#### 6.6.1 内置脚本

**需求1**：修改文档6的 num 字段，+1。

* 插入数据

```json
PUT /test_index/_doc/6
{
    "num": 0,
    "tags": []
}
```

* 执行脚本操作

```json
POST /test_index/_doc/6/_update
{
    // 获取全局上下文ctx
    "script" : "ctx._source.num+=1"
}
```

* 查询数据

```json
GET /test_index/_doc/6

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "6",
    "_version" : 2,
    "_seq_no" : 23,
    "_primary_term" : 1,
    "found" : true,
    "_source" : {
        "num" : 1,
        "tags" : [ ]
    }
}
```

**需求2**：搜索所有文档，将 num 字段乘以2输出

* 插入数据

```json
PUT /test_index/_doc/7
{
    "num": 5
}
```

* 查询

```json
GET /test_index/_search
{
    "script_fields": {
        "my_doubled_field": {
            "script": {
                "lang": "expression",
                "source": "doc['num'] * multiplier",
                "params": {
                    "multiplier": 2
                }
            }
        }
    }
}
```

* 返回

```json
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "7",
    "_score" : 1.0,
    "fields" : {
        "my_doubled_field" : [
            10.0
        ]
    }
}
```

#### 6.6.2 外部脚本

Painless 是内置支持的。脚本内容可以通过多种途径传给 es，包括 rest 接口，或者放到 config/scripts 目录等，默认开启。

注意：脚本性能低下，且容易发生注入，本教程忽略。

官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html

### 6.7 ES的并发问题

如同秒杀，多线程情况下，es同样会出现并发冲突问题

#### 6.7.1 悲观锁与乐观锁机制

为控制并发问题，我们通常采用锁机制。分为悲观锁和乐观锁两种机制。

**悲观锁**：

* 很悲观，所有情况都上锁。此时只有一个线程可以操作数据。具体例子为数据库中的行级锁、表级锁、读锁、写锁等。

* 特点：优点是方便，直接加锁，对程序透明。缺点是效率低。

**乐观锁**：

* 很乐观，对数据本身不加锁。提交数据时，通过一种机制验证是否存在冲突，如es中通过版本号验证。

* 特点：优点是并发能力高。缺点是操作繁琐，在提交数据时，可能反复重试多次。

#### 6.7.2 ES内部基于_version乐观锁控制

**实验基于 _version 的版本控制**：es 对于文档的增删改都是基于版本号。

1. 新增多次文档：

```json
PUT /test_index/_doc/3
{
    "test_field": "test"
}

// 返回：返回版本号递增
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "3",
    "_version" : 1,
    "result" : "created",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 8,
    "_primary_term" : 1
}
```

2. 删除此文档

```json
DELETE /test_index/_doc/3

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "3",
    "_version" : 2,
    "result" : "deleted",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 9,
    "_primary_term" : 1
}
```

3. 再新增

```json
PUT /test_index/_doc/3
{
    "test_field": "test"
}

// 返回：
{
    "_index" : "test_index",
    "_type" : "_doc",
    "_id" : "3",
    "_version" : 3,    # 可以看到version变成了3
    "result" : "created",
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "_seq_no" : 10,
    "_primary_term" : 1
}
```

可以看到版本号依然递增，验证延迟删除策略。

如果删除一条数据立马删除的话，所有分片和副本都要立马删除，对 es 集群压力太大。

#### 6.7.3 ES内部并发控制

es内部主从同步时，是多线程异步。乐观锁机制。

**演示客户端程序基于 _version 并发操作流程**

* **新建文档**

  ```json
  PUT /test_index/_doc/3
  {
      "test_field": "itcast"
  }
  
  // 返回： 
  {
      "_index" : "test_index",
      "_type" : "_doc",
      "_id" : "3",
      "_version" : 1,
      "result" : "created",
      "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
      },
      "_seq_no" : 12,
      "_primary_term" : 1
  }
  ```

* **客户端1修改，带版本号1**

  首先获取数据的当前版本号

  ```json
  GET /test_index/_doc/3
  
  // 返回：
  {
      "_index" : "test_index",
      "_type" : "_doc",
      "_id" : "3",
      "_version" : 1,
      "_seq_no" : 12,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
          "test_field" : "itcast"
      }
  }
  ```

  更新文档

  ```json
  PUT /test_index/_doc/3?version=1
  {
    "test_field": "itcast1"
  }
  // 提示更新失败：Validation Failed: 1: internal versioning can not be used for optimistic concurrency control. Please use `if_seq_no` and `if_primary_term` instead;
  
  // 正确的更新方式：要使用if_seq_no+if_primary_term，查询的时候会有体现
  PUT /test_index/_doc/3?if_seq_no=12&if_primary_term=1
  {
    "test_field": "itcast1"
  }
  // 更新成功
  {
      "_index" : "test_index",
      "_type" : "_doc",
      "_id" : "3",
      "_version" : 2,
      "result" : "updated",
      "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
      },
      "_seq_no" : 13,
      "_primary_term" : 1
  }
  ```

* **客户端2并发修改，带版本号**

  ```json
  PUT /test_index/_doc/5?if_seq_no=21&if_primary_term=1
  {
  	"test_field": "itcast1"
  }
  // 更新失败，因为版本号升级了："reason" : "[3]: version conflict, required seqNo [12], primary term [1]. current document has seqNo [13] and primary term [1]
  ```

* **客户端2重新查询，得到最新版本为2，seq_no=13**

  ```json
  GET /test_index/_doc/4
  
  // 返回：
  {
      "_index" : "test_index",
      "_type" : "_doc",
      "_id" : "3",
      "_version" : 2,
      "_seq_no" : 13,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
          "test_field" : "itcast1"
      }
  }
  ```

* **客户端2并发修改，带版本号2**

  ```json
  PUT /test_index/_doc/5?if_seq_no=13&if_primary_term=1
  {
    "test_field": "itcast2"
  }
  
  // 此时修改成功，返回：
  {
      "_index" : "test_index",
      "_type" : "_doc",
      "_id" : "3",
      "_version" : 3,
      "result" : "updated",
      "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
      },
      "_seq_no" : 14,
      "_primary_term" : 1
  }
  ```

**演示自己手动控制版本号 external version**

* 背景：已有数据是在数据库中，有自己手动维护的版本号的情况下，可以使用 external version 控制。

* 要求：修改时 external version 要**大于**当前文档的 _version

* 对比：基于 _version 时，修改的文档 _version 等于当前文档的版本号，使用：`?version=1&version_type=external`

* **新建文档**

  ```json
  PUT /test_index/_doc/4
  {
      "test_field": "itcast"
  }
  
  // 返回：
  {
      "_index" : "test_index",
      "_type" : "_doc",
      "_id" : "4",
      "_version" : 1,
      "result" : "created",
      "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
      },
      "_seq_no" : 16,
      "_primary_term" : 1
  }
  ```

* **客户端1修改文档**

  ```json
  PUT /test_index/_doc/4?version=2&version_type=external
  {
  	"test_field": "itcast1"
  }
  
  // 修改成功，返回：
  {
      "_index" : "test_index",
      "_type" : "_doc",
      "_id" : "4",
      "_version" : 2,
      "result" : "updated",
      "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
      },
      "_seq_no" : 17,
      "_primary_term" : 1
  }
  ```

* **客户端2同时修改**

  ```json
  PUT /test_index/_doc/4?version=2&version_type=external
  {
  	"test_field": "itcast2"
  }
  
  // 修改失败，返回：
  {
      "error" : {
          "root_cause" : [
              {
                  "type" : "version_conflict_engine_exception",
                  "reason" : "[4]: version conflict, current version [2] is higher or equal to the one provided [2]",
                  "index_uuid" : "GVu-xz-jQ66VSsaqsP2PIw",
                  "shard" : "0",
                  "index" : "test_index"
              }
          ],
          "type" : "version_conflict_engine_exception",
          "reason" : "[4]: version conflict, current version [2] is higher or equal to the one provided [2]",
          "index_uuid" : "GVu-xz-jQ66VSsaqsP2PIw",
          "shard" : "0",
          "index" : "test_index"
      },
      "status" : 409
  }
  ```

* **客户端2重新查询数据**

  ```json
  GET /test_index/_doc/4
  
  // 返回：
  {
      "_index" : "test_index",
      "_type" : "_doc",
      "_id" : "4",
      "_version" : 2,
      "_seq_no" : 16,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
          "test_field" : "itcast1"
      }
  }
  ```

* **客户端2重新修改数据，成功修改**

  ```json
  PUT /test_index/_doc/4?version=3&version_type=external
  {
  	"test_field": "itcast2"
  }
  ```

#### 6.7.4 retry_on_conflict参数

**retry_on_conflict**：指定重试次数

```json
POST /test_index/_doc/5/_update?retry_on_conflict=3
{
    "doc": {
        "test_field": "itcast1"
    }
}
```

**与 _version结合使用**

```json
POST /test_index/_doc/5/_update?retry_on_conflict=3&version=22&version_type=external
{
    "doc": {
        "test_field": "itcast1"
    }
}
```

### 6.8 批量查询mget

单条查询 `GET /test_index/_doc/1`，如果查询多个 id 的文档一条一条查询，网络开销太大。

**mget 批量查询**

```json
GET /_mget
{
   "docs" : [
      {
         "_index" : "test_index",
         "_type" :  "_doc",
         "_id" :    1
      },
      {
         "_index" : "test_index",
         "_type" :  "_doc",
         "_id" :    7
      }
   ]
}
```

返回：

```json
#! Deprecation: [types removal] Specifying types in multi get requests is deprecated.
{
    "docs" : [
        {
            "_index" : "test_index",
            "_type" : "_doc",
            "_id" : "2",
            "_version" : 6,
            "_seq_no" : 12,
            "_primary_term" : 1,
            "found" : true,
            "_source" : {
                "test_field" : "test12333123321321"
            }
        },
        {
            "_index" : "test_index",
            "_type" : "_doc",
            "_id" : "3",
            "_version" : 6,
            "_seq_no" : 18,
            "_primary_term" : 1,
            "found" : true,
            "_source" : {
                "test_field" : "test3213"
            }
        }
    ]
}
```

提示去掉 `type：Deprecation: [types removal] Specifying types in multi get requests is deprecated.`

```json
GET /_mget
{
   "docs" : [
      {
         "_index" : "test_index",
         "_id" :    2
      },
      {
         "_index" : "test_index",
         "_id" :    3
      }
   ]
}
```

**同一索引下批量查询：**

```json
GET /test_index/_mget
{
   "docs" : [
      {
         "_id" :    2
      },
      {
         "_id" :    3
      }
   ]
}
```

**第三种写法：搜索写法**

```json
GET /test_index/_search
{
    "query": {
        "ids" : {
            "values" : ["1", "7"]
        }
    }
}
```

### 6.9 批量增删改bulk

Bulk 操作解释将文档的增删改查一些系列操作，通过一次请求全都做完。减少网络传输次数。

语法：

```json
POST /_bulk
{"action": {"metadata"}}
{"data"}
```

如下操作：删除5，新增14，修改2。

```json
POST /_bulk
{ "delete": { "_index": "test_index",  "_id": "5" }} 
{ "create": { "_index": "test_index",  "_id": "14" }}
{ "test_field": "test14" }
{ "update": { "_index": "test_index",  "_id": "2"} }
{ "doc" : {"test_field" : "bulk test"} }
```

**总结：**

1. 功能：
   1. delete：删除一个文档，只要 1 个 json 串就可以了
   2. create：相当于强制创建  `PUT /index/type/id/_create` 
   3. index：普通的 put 操作，可以是创建文档，也可以是全量替换文档
   4. update：执行的是局部更新 partial update 操作

2. 格式：每个 json 不能换行。相邻 json 必须换行。

3. 隔离：每个操作互不影响。操作失败的行会返回其失败信息。

4. 实际用法：bulk 请求一次不要太大，否则一下积压到内存中，性能会下降。所以，一次请求几千个操作、大小在几M正好。

### 6.15 document总结

**章节回顾**

1. 文档的增删改查

2. 文档字段解析

3. 内部锁机制

4. 批量查询修改

**es **：一个分布式的文档数据存储系统 distributed document store；es 看做一个分布式 nosql 数据库，如 redis\mongoDB\hbase。

文档数据：es 可以存储和操作 json 文档类型的数据，而且这也是 es 的核心数据结构。

存储系统：es 可以对 json 文档类型的数据进行存储，查询，创建，更新，删除，等等操作。

**应用场景**

- 大数据。es 的分布式特点，水平扩容承载大数据。
- 数据结构灵活。列随时变化。使用关系型数据库将会建立大量的关联表，增加系统复杂度。
- 数据操作简单。就是查询，不涉及事务。

**举例**

电商页面、传统论坛页面等，面向的对象比较复杂，但是作为终端，没有太复杂的功能（事务），只涉及简单的增删改查 crud。

这个时候选用 ES 这种 NoSQL 型的数据存储，比传统的复杂的事务强大的关系型数据库，更加合适一些，无论是性能，还是吞吐量，可能都会更好。

## 7. Java API 实现文档管理

### 7.1 ES技术特点

1. es技术比较特殊，不像其他分布式、大数据课程，haddop、spark、hbase；es 代码层面很好写，难的是概念的理解。

2. es最重要的是他的 rest api，跨语言的，在真实生产中，探查数据、分析数据，使用 rest 更方便。

3. 本教程将会大量讲解内部原理及 rest api；java 代码会在重要的 api 后学习。

### 7.2 Java客户端简单获取数据

Java API文档：https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html

[Java Low Level REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-low.html)：偏向底层

[Java High Level REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high.html)：高级封装

**导包**

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.3.0</version>
    <exclusions>
        <!-- 下方重新导入es的包，以求和安装的es版本相同 -->
        <exclusion>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>7.3.0</version>
</dependency>
```

**代码步骤 **

```java
// 1.获取连接客户端
RestHighLevelClient client = new RestHighLevelClient(
    RestClient.builder(new HttpHost("localhost", 9200, "http")));
// 2.构建请求
GetRequest getRequest = new GetRequest("book", "1");
// 3.执行
GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
// 4.获取结果
if (getResponse.isExists()) {
    long version = getResponse.getVersion();
    String sourceAsString = getResponse.getSourceAsString();//检索文档(String形式)
    System.out.println(sourceAsString);
}
```

### 7.3 结合SpringBoot测试文档查询

使用spring boot test理由：当今趋势，方便开发，创建连接交由 spring 容器，避免每次请求的网络开销。

```xml
<dependencies>
    <!--es客户端-->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.3.0</version>
        <exclusions>
            <exclusion>
                <groupId>org.elasticsearch</groupId>
                <artifactId>elasticsearch</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.8.0</version>
    </dependency>
	
    <!--springboot依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <version>2.0.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <version>2.0.6.RELEASE</version>
    </dependency>
</dependencies>
```

**配置**：`application.yml`

```yaml
spring:
  application:
    name: service-search
heima:
  elasticsearch:
    hostlist: 127.0.0.1:9200  # 多个结点中间用逗号分隔
```

> 日志配置文件：logback-spring.xml
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> 
> <configuration>
>        <!--定义日志文件的存储地址,使用绝对路径-->
>        <property name="LOG_HOME" value="d:/logs"/>
> 
>        <!-- Console 输出设置 -->
>        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
>            <encoder>
>                <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
>                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
>                <charset>utf8</charset>
>            </encoder>
>        </appender>
> 
>        <!-- 按照每天生成日志文件 -->
>        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
>            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
>                <!--日志文件输出的文件名-->
>                <fileNamePattern>${LOG_HOME}/xc.%d{yyyy-MM-dd}.log</fileNamePattern>
>            </rollingPolicy>
>            <encoder>
>                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
>            </encoder>
>        </appender>
> 
>        <!-- 异步输出 -->
>        <appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
>            <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
>            <discardingThreshold>0</discardingThreshold>
>            <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
>            <queueSize>512</queueSize>
>            <!-- 添加附加的appender,最多只能添加一个 -->
>            <appender-ref ref="FILE"/>
>        </appender>
> 
>     <logger name="org.apache.ibatis.cache.decorators.LoggingCache" level="DEBUG" additivity="false">
>            <appender-ref ref="CONSOLE"/>
>        </logger>
>        
>        <logger name="org.springframework.boot" level="DEBUG"/>
>        <root level="info">
>            <!--<appender-ref ref="ASYNC"/>-->
>            <appender-ref ref="FILE"/>
>            <appender-ref ref="CONSOLE"/>
>        </root>
> </configuration>
> ```

**代码**

1. 主类

```java
@SpringBootApplication
public class SearchApplication {
    public static void main(String[] args) {
        SpringApplication.run(SearchApplication.class,args);
    }
}
```

2. 配置类

```java
@Configuration
public class ElasticsearchConfig {
    
    @Value("${heima.elasticsearch.hostlist}")
    private String hostlist;

    @Bean(destroyMethod = "close")
    public RestHighLevelClient restHighLevelClient(){
        String[] split = hostlist.split(",");
        HttpHost[] httpHostsArray = new HttpHost[split.length];
        for (int i = 0; i < split.length; i++) {
            String item = split[i];
            httpHostsArray[i] = new HttpHost(item.split(":")[0],Integer.parseInt(item.split(":")[1]),"http");
        }
        return new RestHighLevelClient(RestClient.builder(httpHostsArray));
    }
}
```

3. 测试类

```java
@Autowired
RestHighLevelClient client;

@Test
public void testGet() throws IOException {
    // 1.构建请求
    GetRequest getRequest = new GetRequest("test_post", "1");

    // ========================可选参数 start======================
    // 为特定字段配置_source_include
    String[] includes = new String[]{"user", "message"};
    String[] excludes = Strings.EMPTY_ARRAY;
    FetchSourceContext fetchSourceContext = new FetchSourceContext(true, includes, excludes);
    getRequest.fetchSourceContext(fetchSourceContext);

    // 为特定字段配置_source_excludes
    String[] includes1 = Strings.EMPTY_ARRAY;
    String[] excludes1 = new String[]{"user", "message"};
    FetchSourceContext fetchSourceContext1 = new FetchSourceContext(true, includes1, excludes1);
    getRequest.fetchSourceContext(fetchSourceContext1);
    // ========================可选参数 end=====================

    // ========================设置路由 start=====================
    getRequest.routing("routing");
    // ========================设置路由 end=====================
    
    // 查询 同步查询
    GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);

    // 获取结果
    if (getResponse.isExists()) {
        long version = getResponse.getVersion();
        // 检索文档(String形式)
        String sourceAsString = getResponse.getSourceAsString();
        System.out.println(sourceAsString);
        // 以字节接受
        byte[] sourceAsBytes = getResponse.getSourceAsBytes();
        // 用map的方式
        Map<String, Object> sourceAsMap = getResponse.getSourceAsMap();
        System.out.println(sourceAsMap);
    }

    // 异步查询
    // ActionListener<GetResponse> listener = new ActionListener<GetResponse>() {
    //     // 查询成功时的立马执行的方法
    //     @Override
    //     public void onResponse(GetResponse getResponse) {
    //         long version = getResponse.getVersion();
    //		   // 检索文档(String形式)
    //         String sourceAsString = getResponse.getSourceAsString();
    //         System.out.println(sourceAsString);
    //     }
    //
    //     // 查询失败时的立马执行的方法
    //     @Override
    //     public void onFailure(Exception e) {
    //         e.printStackTrace();
    //     }
    // };
    // // 执行异步请求
    // client.getAsync(getRequest, RequestOptions.DEFAULT, listener);
    // try {
    //     Thread.sleep(5000);
    // } catch (InterruptedException e) {
    //     e.printStackTrace();
    // }
}
```

### 7.4 结合 SpringBoot 测试文档新增

Rest API：

```json
PUT test_post/_doc/2
{
    "user":"tomas",
    "postDate":"2019-07-18",
    "message":"trying out es1"
}
```

对应代码：

```java
@Test
public void testAdd() throws IOException {
    // 1. 构建请求
    IndexRequest request=new IndexRequest("test_posts");
    // 设置id
    request.id("3");
    // =======================构建文档============================
    // 构建方法1
    String jsonString="{\n" +
        "  \"user\":\"tomas J\",\n" +
        "  \"postDate\":\"2019-07-18\",\n" +
        "  \"message\":\"trying out es3\"\n" +
        "}";
    // 构建请求体
    request.source(jsonString, XContentType.JSON);

    // 构建方法2
    // Map<String, Object> jsonMap = new HashMap<>();
    // jsonMap.put("user", "tomas");
    // jsonMap.put("postDate", "2019-07-18");
    // jsonMap.put("message", "trying out es2");
    // request.source(jsonMap);

    // 构建方法3
    // XContentBuilder builder= XContentFactory.jsonBuilder();
    // builder.startObject();
    // {
    //     builder.field("user", "tomas");
    //     builder.timeField("postDate", new Date());
    //     builder.field("message", "trying out es2");
    // }
    // builder.endObject();
    // request.source(builder);

    // 构建方法4
    // request.source("user","tomas",
    //                "postDate",new Date(),
    //                "message", "trying out es2");
    //
    // =======================构建文档============================

    // =======================可选参数============================
    // 设置超时时间
    // request.timeout(TimeValue.timeValueSeconds(1));
    // request.timeout("1s");

    // 自己维护版本号
    // request.version(2);
    // request.versionType(VersionType.EXTERNAL);



    // 2. 执行同步方式
    IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);
    // 异步方式
    // ActionListener<IndexResponse> listener=new ActionListener<IndexResponse>() {
    //     @Override
    //     public void onResponse(IndexResponse indexResponse) {
    //
    //     }
    //
    //     @Override
    //     public void onFailure(Exception e) {
    //
    //     }
    // };
    // client.indexAsync(request,RequestOptions.DEFAULT, listener );
    // try {
    //     Thread.sleep(5000);
    // } catch (InterruptedException e) {
    //     e.printStackTrace();
    // }

    // 3.获取结果
    String index = indexResponse.getIndex();
    String id = indexResponse.getId();
    // 获取插入的类型
    if(indexResponse.getResult()== DocWriteResponse.Result.CREATED){
        DocWriteResponse.Result result=indexResponse.getResult();
        System.out.println("CREATED:"+result);
    }else if(indexResponse.getResult()== DocWriteResponse.Result.UPDATED){
        DocWriteResponse.Result result=indexResponse.getResult();
        System.out.println("UPDATED:"+result);
    }

    ReplicationResponse.ShardInfo shardInfo = indexResponse.getShardInfo();
    if(shardInfo.getTotal() != shardInfo.getSuccessful()){
        System.out.println("处理成功的分片数少于总分片！");
    }
    if(shardInfo.getFailed()>0){
        for (ReplicationResponse.ShardInfo.Failure failure:shardInfo.getFailures()) {
            String reason = failure.reason();  // 处理潜在的失败原因
            System.out.println(reason);
        }
    }
}
```

### 7.5 结合 SpringBoot 测试文档修改

Rest API：

```json
post /test_posts/_doc/3/_update 
{
   "doc": {
      "user"："tomas J"
   }
}
```

对应代码：

```java
@Test
public void testUpdate() throws IOException {
    // 1.构建请求
    UpdateRequest request = new UpdateRequest("test_posts", "3");
    Map<String, Object> jsonMap = new HashMap<>();
    jsonMap.put("user", "tomas JJ");
    request.doc(jsonMap);
    // ========================可选参数===============================
    // 超时时间
    request.timeout("1s");
    // 重试次数
    request.retryOnConflict(3);

    // 设置在继续更新之前，必须激活的分片数
    request.waitForActiveShards(2);
    // 所有分片都是active状态，才更新
    request.waitForActiveShards(ActiveShardCount.ALL);

    // 2.执行
    // 同步方式
    UpdateResponse updateResponse = client.update(request, RequestOptions.DEFAULT);

    // 3.获取数据
    updateResponse.getId();
    updateResponse.getIndex();

    // 判断结果
    if (updateResponse.getResult() == DocWriteResponse.Result.CREATED) {
        DocWriteResponse.Result result = updateResponse.getResult();
        System.out.println("CREATED:" + result);
    } else if (updateResponse.getResult() == DocWriteResponse.Result.UPDATED) {
        DocWriteResponse.Result result = updateResponse.getResult();
        System.out.println("UPDATED:" + result);
    }else if(updateResponse.getResult() == DocWriteResponse.Result.DELETED){
        DocWriteResponse.Result result = updateResponse.getResult();
        System.out.println("DELETED:" + result);
    }else if (updateResponse.getResult() == DocWriteResponse.Result.NOOP){
        // 没有操作
        DocWriteResponse.Result result = updateResponse.getResult();
        System.out.println("NOOP:" + result);
    }
}
```

### 7.6 结合 SpringBoot 测试文档删除

```json
DELETE /test_posts/_doc/3
```

代码：

```java
@Test
public void testDelete() throws IOException {
    // 1.构建请求
    DeleteRequest request =new DeleteRequest("test_posts","3");
    // 2.执行
    DeleteResponse deleteResponse = client.delete(request, RequestOptions.DEFAULT);

    // 3.获取数据
    deleteResponse.getId();
    deleteResponse.getIndex();
    DocWriteResponse.Result result = deleteResponse.getResult();
    System.out.println(result);
}
```

### 7.7 结合 SpringBoot 测试文档bulk

Rest API：

```json
POST /_bulk
{"action": {"metadata"}}
{"data"}
```

对应代码：

```java
@Test
public void testBulk() throws IOException {
    // 1.创建请求
    BulkRequest request = new BulkRequest();

    request.add(new IndexRequest("post").id("1").source(XContentType.JSON, "field", "1"));
    request.add(new IndexRequest("post").id("2").source(XContentType.JSON, "field", "2"));
    request.add(new UpdateRequest("post","2").doc(XContentType.JSON, "field", "3"));
    request.add(new DeleteRequest("post").id("1"));

    // 2.执行
    BulkResponse bulkResponse = client.bulk(request, RequestOptions.DEFAULT);

    // 3.获取结果
    for (BulkItemResponse itemResponse : bulkResponse) {
        DocWriteResponse response = itemResponse.getResponse();
        switch (itemResponse.getOpType()){
            case INDEX:
                IndexResponse indexResponse = (IndexResponse) response;
                System.out.println("INDEX:"+indexResponse.getResult());
                break;
            case CREATE:
                IndexResponse createResponse = (IndexResponse) response;
                System.out.println("CREATE:"+createResponse.getResult());
                break;
            case UPDATE:
                UpdateResponse updateResponse = (UpdateResponse) response;
                System.out.println("UPDATE:"+updateResponse.getResult());
                break;
            case DELETE:
                DeleteResponse deleteResponse = (DeleteResponse) response;
                System.out.println("DELETE:"+deleteResponse.getResult());
                break;
        }
    }
}
```

