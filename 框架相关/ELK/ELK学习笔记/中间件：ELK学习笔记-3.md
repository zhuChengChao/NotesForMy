# 中间件：ELK学习笔记-3

> 很早之前粗略的学习过Lucene和Elasticsearch，然后最近实习过程中需要用到一些ES的基础操作，虽说使用起来没啥问题，但还是想稍微系统的学习一下，也能在秋招的时候稍微说道说道。
>
> 在 bilibili 上搜了一个课程，课程地址：[黑马：ELK高级搜索](https://www.bilibili.com/video/BV1Nt4y1m7qL)，跟着视频也做了些笔记，总的分了7篇，此为3/7篇。
>
> > 学的很草率...空了再来完善:cry:

## 8. ES内部机制

### 8.1 ES分布式基础

**es 对复杂分布式机制的透明隐藏特性**

-   分布式机制：分布式数据存储及共享。
-   分片机制：数据存储到哪个分片，副本数据写入。
-   集群发现机制：cluster discovery，新启动 es 实例，自动加入集群。
-   shard 负载均衡：大量数据写入及查询，es 会将数据平均分配。
-   shard 副本：新增副本数，分片重分配。

**es 的垂直扩容与水平扩容**

* **垂直扩容**：使用更加强大的服务器替代老服务器，但单机存储及运算能力有上线，且成本直线上升，如 10t 服务器1万，单个 10T 服务器可能20万。

* **水平扩容**：采购更多服务器，加入集群。

**增减或减少节点时的数据 rebalance**：新增或减少es实例时，es集群会将数据重新分配。

**master 节点**

-  创建删除节点
-  创建删除索引

**节点对等的分布式架构**

- 节点对等，每个节点都能接收所有的请求
- 自动请求路由
- 响应收集

### 8.2 ES的shard&replica机制

**shard&replica机制**

1. 每个 index 包含一个或多个 shard

2. 每个 shard 都是一个最小工作单元，承载部分数据，是一个 Lucene 实例，完整的建立索引和处理请求的能力

3. 增减节点时，shard 会自动在 nodes 中负载均衡

4. primary shard 和 replica shard，每个 document 肯定只存在于某一个 primary shard 以及其对应的 replica shard 中，不可能存在于多个 primary shard

5. replica shard 是 primary shard 的副本，负责容错，以及承担读请求负载

6. primary shard 的数量在创建索引的时候就固定了，replica shard 的数量可以随时修改

7. primary shard 的默认数量是1，replica 默认是1，即：默认共有2个shard，1个primary shard，1个replica shard

   > 注意：es7以前primary shard的默认数量是5，replica默认是1，默认有10个shard，5个primary shard，5个replica shard

8. primary shard 不能和自己的 replica shard 放在同一个节点上（否则节点宕机，primary shard 和 replica shard 都丢失，起不到容错的作用），但是可以和其他 primary shard 的 replica shard 放在同一个节点上

**单node下创建index**

1. 单 node 环境下，创建一个 index，设置为：有3个 primary shard，3个 replica shard
2. 集群 status 是 yellow
3. 这个时候，只会将3个 primary shard 分配到仅有的一个 node 上去，另外3个replica shard是无法分配
4. 集群可以正常工作，但是一旦出现节点宕机，数据全部丢失，而且集群不可用，无法承接任何请求

```json
PUT /test_index
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```

**2个node下replica shard的分配**

1. replica shard 分配：3 个 primary shard，3 个 replica shard，1 node
2. 写情况：primary → replica同步
3. 读请求：primary/replica

### 8.3 横向扩容

- 分片自动负载均衡，分片向空闲机器转移。
- 每个节点存储更少分片，系统资源给与每个分片的资源更多，整体集群性能提高。
- 扩容极限：节点数大于整体分片数，则必有空闲机器。
- 超出扩容极限时，可以增加副本数，如设置副本数为2，总共3*3=9个分片。9台机器同时运行，存储和搜索性能更强。容错性更好。
- 容错性：只要一个索引的所有主分片在，集群就就可以运行。

### 8.6 ES容错机制

容错机制：master选举 replica容错，数据恢复

以3分片，2副本数，3节点为例介绍：

- master node 宕机，自动 master 选举，集群状态为 red
- replica shard 容错：新 master 将 replica shard 提升为 primary shard，集群状态为 yellow
- 重启宕机 node，master copy replica 到该 node，使用原有的 shard 并同步宕机后的修改，green

## 9. 文档存储机制

### 9.1 数据路由

**文档存储路由到相应分片**：一个文档，最终会落在主分片的一个分片上，到底应该在哪一个分片？这就是数据路由。

**路由算法**

* 哈希值对主分片数取模：`shard = hash(routing) % number_of_primary_shards`

* 举例：
  * 对一个文档进行 crud 时，都会带一个路由值 routing number，默认为文档 _id（可能是手动指定，也可能是自动生成）。
  * 存储 1 号文档，经过哈希计算，哈希值为2，此索引有3个主分片，那么计算 2%3=2，就算出此文档在 P2 分片上。
  * 决定一个 document 在哪个 shard 上，最重要的一个值就是 routing 值，默认是 _id，也可以手动指定，相同的 routing 值，每次过来，从 hash 函数中，产出的hash值一定是相同的。
  * 无论 hash 值是几，无论是什么数字，对 number_of_primary_shards 求余数，结果一定是在 0~number_of_primary_shards-1 之间这个范围内的。

**手动指定 routing number**

```json
PUT /test_index/_doc/15?routing=num
{
  "num": 0,
  "tags": []
}
```

场景：在程序中，架构师可以手动指定已有数据的一个属性为路由值，好处是可以定制一类文档数据存储到一个分片中。缺点是设计不好，会造成数据倾斜。

所以，不同文档尽量放到不同的索引中。剩下的事情交给es集群自己处理。

**主分片数量不可变**：涉及到以往数据的查询搜索，所以一旦建立索引，主分片数不可变。

### 9.2 文档的增删改内部机制

增删改可以看做 update，都是对数据的改动。一个改动请求发送到 es 集群，经历以下四个步骤：

1. 客户端选择一个 node 发送请求过去，这个 node 就是 **coordinating node（协调节点）**

2. coordinating node，对 document 进行路由，将请求转发给对应的 node（有primary shard）

3. 实际的 node 上的 primary shard 处理请求，然后将数据同步到 replica node。

4. coordinating node，如果发现 primary node 和所有 replica node 都搞定之后，就返回响应结果给客户端。

### 9.3 文档的查询内部机制

1. 客户端发送请求到任意一个 node，成为 coordinate node

2. coordinate node 对 document 进行路由，将请求转发到对应的 node，此时会使用 **round-robin 随机轮询算法**，在 primary shard 以及其所有 replica 中随机选择一个，让读请求负载均衡

3. 接收请求的 node 返回 document 给 coordinate node

4. coordinate node 返回 document 给客户端

**特殊情况**：document 如果还在建立索引过程中，可能只有 primary shard 有，任何一个 replica shard 都没有，此时可能会导致无法读取到 document，但是 document 完成索引建立之后，primary shard 和 replica shard 就都有了。

### 9.4. bulk API奇特json格式

```json
// bulk的格式：
POST /_bulk
{"action": {"meta"}}
{"data"}
{"action": {"meta"}}
{"data"}

// 如果用这种格式所带来的问题，后续分析（比较人性化的 Json 格式）
[
    {
        "action":{
            "method":"create"
        },
        "data":{
            "id":1,
            "field1":"java",
            "field1":"spring",
        }
    },
    {
        "action":{
            "method":"create"
        },
        "data":{
            "id":2,
            "field1":"java",
            "field1":"spring",
        }
    }       
]
```

1. bulk 中的每个操作都可能要转发到不同的 node 的 shard 去执行

2. 如果采用比较良好的 json 数组格式，允许任意的换行，整个可读性非常棒，读起来很爽，es 拿到那种标准格式的 json 串以后，要按照下述流程去进行处理
   1. 将 json 数组解析为 **JSONArray 对象**，这个时候，整个数据，就会在内存中出现一份一模一样的拷贝，一份数据是 json 文本，一份数据是 JSONArray 对象
   2. 解析 json 数组里的每个 json，对每个请求中的 document 进行路由
   3. 为路由到同一个 shard 上的多个请求，创建一个请求数组。比如 100 请求中有 10 个是到 P1 shard
   4. 将这个请求数组序列化
   5. 将序列化后的请求数组发送到对应的节点上去

3. 问题：**耗费更多内存，更多的 JVM GC 开销**

   我们之前提到过 bulk size 最佳大小的那个问题，一般建议说在几千条那样，然后大小在 10MB 左右，所以说，可怕的事情来了。假设说现在 100 个 bulk 请求发送到了一个节点上去，然后每个请求是 10MB，100 个请求，就是 1000MB = 1GB，然后每个请求的 json 都 copy 一份为 jsonarray 对象，此时内存中的占用就会翻倍，就会占用 2GB 的内存，甚至还不止。因为弄成 JSONArray 之后，还可能会多搞一些其他的数据结构，2GB+ 的内存占用。

   占用更多的内存可能就会积压其他请求的内存使用量，比如说最重要的搜索请求，分析请求，等等，此时就可能会导致其他请求的性能急速下降。

   另外的话，占用内存更多，就会导致 java 虚拟机的垃圾回收次数更多，更频繁，每次要回收的垃圾对象更多，耗费的时间更多，导致 es 的 java 虚拟机停止工作线程的时间更多。

4. 现在的奇特格式

   ```json
   POST /_bulk
   { "delete": { "_index": "test_index",  "_id": "5" }} \n
   { "create": { "_index": "test_index",  "_id": "14" }}\n
   { "test_field": "test14" }\n
   { "update": { "_index": "test_index",  "_id": "2"} }\n
   { "doc" : {"test_field" : "bulk test"} }\n
   ```

   不用将其转换为 json 对象，不会出现内存中的相同数据的拷贝，**直接按照换行符切割 json**

   对每两个一组的 json，读取 meta，进行 document 路由

   直接将对应的 json 发送到 node 上去

   **最大的优势在于，不需要将json数组解析为一个 JSONArray 对象，形成一份大数据的拷贝，浪费内存空间，尽可能地保证性能。**

## 10. mapping映射入门

### 10.1 mapping映射概念

概念：自动或手动为 index 中的 _doc 建立的一种数据结构和相关配置，简称为 mapping 映射。

插入几条数据，让 es 自动为我们建立一个索引

```json
PUT /website/_doc/1
{
    "post_date": "2019-01-01",
    "title": "my first article",
    "content": "this is my first article in this website",
    "author_id": 11400
}

PUT /website/_doc/2
{
    "post_date": "2019-01-02",
    "title": "my second article",
    "content": "this is my second article in this website",
    "author_id": 11400
}

PUT /website/_doc/3
{
    "post_date": "2019-01-03",
    "title": "my third article",
    "content": "this is my third article in this website",
    "author_id": 11400
}
```

对比数据库建表语句

```mysql
create table website(
    post_date date,
    title varchar(50),     
    content varchar(100),
    author_id int(11) 
);
```

**动态映射**：dynamic mapping，自动为我们建立 index，以及对应的 mapping，mapping 中包含了每个 field 对应的数据类型，以及如何分词等设置。

手动映射：我们当然也可以手动在创建数据之前，先创建 index，以及对应的 mapping

```json
GET  /website/_mapping/

// 返回：显示当前mapping信息
{
    "website" : {
        "mappings" : {
            "properties" : {
                "author_id" : {  // 属性1
                    "type" : "long"
                },
                "content" : {  // 属性2
                    "type" : "text",
                    "fields" : {
                        "keyword" : {
                            "type" : "keyword",
                            "ignore_above" : 256
                        }
                    }
                },
                "post_date" : {  // 属性3
                    "type" : "date"
                },
                "title" : {  // 属性4
                    "type" : "text",
                    "fields" : {
                        "keyword" : {
                            "type" : "keyword",
                            "ignore_above" : 256
                        }
                    }
                }
            }
        }
    }
}
```

尝试各种搜索

```json
GET /website/_search?q=2019                   # 1条结果          
GET /website/_search?q=2019-01-01             # 1条结果
GET /website/_search?q=post_date:2019-01-01   # 1条结果
GET /website/_search?q=post_date:2019         # 1条结果
GET /website/_search?q=website				  # 3条结果
GET /website/_search?q=title:third            # 1条结果
```

搜索结果为什么不一致，因为 es 自动建立 mapping 的时候，为不同的 field 不同的 data type。不同的 data type 的分词、搜索等行为是不一样的。所以出现了 _all field 和 post_date field 的搜索表现完全不一样。

### 10.2 精确匹配与全文搜索的对比分析

**exact value 精确匹配**

`2019-01-01`，exact value，搜索的时候，必须输入 `2019-01-01`，才能搜索出来

如果你输入一个 01，是搜索不出来的

如：`select * from book where name='java'`

**full text 全文检索**

搜“笔记电脑”，笔记本电脑词条会不会出现。

`select * from book where name like '%java%'`

1. 缩写 vs. 全称：cn vs. china

2. 格式转化：like liked likes

3. 大小写：Tom vs tom

4. 同义词：like vs love

`2019-01-01`，`2019 01 01`，搜索 2019，或者 01，都可以搜索出来

* china，搜索 cn，也可以将 china 搜索出来

* likes，搜索 like，也可以将 likes 搜索出来

* Tom，搜索 tom，也可以将 Tom 搜索出来

* like，搜索 love，同义词，也可以将 like 搜索出来

就不是说单纯的只是匹配完整的一个值，而是可以对值进行拆分词语后（分词）进行匹配，也可以通过缩写、时态、大小写、同义词等进行匹配。

### 10.3 全文检索下倒排索引核心原理

* doc1：I really liked my small dogs, and I think my mom also liked them.

* doc2：He never liked any dogs, so I hope that my mom will not expect me to liked him.

**分词，初步的倒排索引的建立**

演示了一下倒排索引最简单的建立的一个过程

| term       | doc1 | doc2 |
| ---------- | ---- | ---- |
| **I**      | √    | √    |
| **really** | √    |      |
| **liked**  | √    | √    |
| **my**     | √    | √    |
| **small**  | √    |      |
| **dogs**   | √    |      |
| **and**    | √    |      |
| **think**  | √    |      |
| **mom**    | √    | √    |
| **also**   | √    |      |
| **them**   | √    |      |
| **He**     |      | √    |
| **never**  |      | √    |
| **any**    |      | √    |
| **so**     |      | √    |
| **hope**   |      | √    |
| **that**   |      | √    |
| **will**   |      | √    |
| **not**    |      | √    |
| **expect** |      | √    |
| **me**     |      | √    |
| **to**     |      | √    |
| **him**    |      | √    |

**搜索**

* mother like little dog，不可能有任何结果

* 但这不是我们想要的结果，同义词 mom\mother 在我们看来是一样，想进行标准化操作。

**重建倒排索引**

* normalization 正规化，建立倒排索引的时候，会执行一个操作，也就是说对拆分出的各个单词进行相应的处理，以提升后面搜索的时候能够搜索到相关联的文档的概率

* 时态的转换，单复数的转换，同义词的转换，大小写的转换

  mom ―> mother、liked ―> like、small ―> little、dogs ―> dog

* 重新建立倒排索引，加入 normalization，再次用 mother liked little dog 搜索，就可以搜索到了

| **word**   | doc1 | doc2 | normalization   |
| ---------- | ---- | ---- | --------------- |
| **I**      | √    | √    |                 |
| **really** | √    |      |                 |
| **like**   | √    | √    | liked ―> like   |
| **my**     | √    | √    |                 |
| **little** | √    |      | small ―> little |
| **dog**    | √    |      | dogs ―> dog     |
| **and**    | √    |      |                 |
| **think**  | √    |      |                 |
| **mother** | √    | √    | mom ―> mother   |
| **also**   | √    |      |                 |
| **them**   | √    |      |                 |
| **He**     |      | √    |                 |
| **never**  |      | √    |                 |
| **any**    |      | √    |                 |
| **so**     |      | √    |                 |
| **hope**   |      | √    |                 |
| **that**   |      | √    |                 |
| **will**   |      | √    |                 |
| **not**    |      | √    |                 |
| **expect** |      | √    |                 |
| **me**     |      | √    |                 |
| **to**     |      | √    |                 |
| **him**    |      | √    |                 |

**重新搜索**

* 搜索：mother liked  little dog，

* 对搜索条件经行分词 normalization

  mother 、liked → like、little 、dog

* 此时doc1和doc2都会搜索出来

### 10.4 分词器analyzer

#### 10.4.1 分词器analyzer介绍

**作用**：

* 切分词语，normalization（提升召回率）

* 给你一段句子，然后将这段句子拆分成一个一个的单个的单词，同时对每个单词进行 normalization（时态转换，单复数转换）

* recall / 召回率：搜索的时候，增加能够搜索到的结果的数量

**analyzer 组成部分：**

1. **character filter 字符过滤器  **：在一段文本进行分词之前，先进行预处理，比如说最常见的就是：过滤 html 标签（`<span>hello<span>` → hello），`&` → and（ `I&you` → I and you）

2. **tokenizer 分词器  **：分词，hello you and me → hello, you, and, me

3. **token filter 词单元过滤器  **：大小写转换 lowercase，去掉停用词 stop word，单词变成词干，同义词转换,如：dogs → dog，liked → like，Tom → tom，a/the/an → 干掉，mother → mom，small → little，

一个分词器，很重要，将一段文本进行各种处理，最后处理好的结果才会拿去建立倒排索引。

#### 10.4.2 内置分词器的介绍

例句：`Set the shape to semi-transparent by calling set_trans(5)`

* **standard analyzer 标准分词器(默认)**：set, the, shape, to, semi, transparent, by, calling, set_trans, 5

* **simple analyzer简单分词器**：set, the, shape, to, semi, transparent, by, calling, set, trans

* **whitespace analyzer 空格分词器**：Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

* **language analyzer（特定的语言的分词器，比如说，english，英语分词器）**：set, shape, semi, transpar, call, set_tran, 5

官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-analyzers.html

![分词器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261416339.png)

### 10.5 query string

**query string 分词**：query string 必须以和 index 建立时相同的 analyzer 进行分词，query string 对 exact value 和 full text 的区别对待，如： 

* date：**exact value 精确匹配**
* text：**full text 全文检索**

**测试分词器**

```json
GET /_analyze
{
    "analyzer": "standard",
    "text": "Text to analyze 80"
}
```

返回值：

```json
{
    "tokens" : [
        {
            "token" : "text",         # token 实际存储的term 关键字
            "start_offset" : 0,       # start_offset/end_offset字符在原始字符串中的位置
            "end_offset" : 4,
            "type" : "<ALPHANUM>",    # 数据类型
            "position" : 0            # position 在此词条在原文本中的位置
        },
        {
            "token" : "to",
            "start_offset" : 5,
            "end_offset" : 7,
            "type" : "<ALPHANUM>",
            "position" : 1
        },
        {
            "token" : "analyze",
            "start_offset" : 8,
            "end_offset" : 15,
            "type" : "<ALPHANUM>",
            "position" : 2
        },
        {
            "token" : "80",
            "start_offset" : 16,
            "end_offset" : 18,
            "type" : "<NUM>",
            "position" : 3
        }
    ]
}
```

### 10.6 mapping回顾总结

1. 往 es 里面直接插入数据，es 会自动建立索引，同时建立对应的 mapping (dynamic mapping)
2. mapping 中就自动定义了每个 field 的数据类型
3. 不同的数据类型（比如说 text 和 date），可能有的是 exact value，有的是 full text
4. exact value：在建立倒排索引的时候，分词的时候，是将整个值一起作为一个关键词建立到倒排索引中的；
5. full text：会经历各种各样的处理，分词，normaliztion（时态转换，同义词转换，大小写转换），才会建立到倒排索引中。
6. 同时，exact value 和 full text 类型的 field 就决定了，在一个搜索过来的时候，对 exact value field 或者是 full text field 进行搜索的行为也是不一样的，会跟建立倒排索引的行为保持一致；比如说 exact value 搜索的时候，就是直接按照整个值进行匹配；full text query string，也会进行分词和 normalization 再去倒排索引中去搜索
7. 可以用 es 的 dynamic mapping，让其自动建立 mapping，包括自动设置数据类型；也可以提前手动创建 index 和 mapping，自己对各个 field 进行设置，包括数据类型，包括索引行为，包括分词器，等。

### 10.7 mapping的数据类型

**核心的数据类型**

* string 类型：text and keyword

* 数字类型：byte，short，integer，long，float，double

* bool 类型：boolean

* 日期类型：date

详见：https://www.elastic.co/guide/en/elasticsearch/reference/7.3/mapping-types.html，下图是 ES7.3 核心的字段类型如下：

![ES核心字段](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261416648.png)

**dynamic mapping 推测规则**

* true or false  →  boolean

* 123  →  long

* 123.45  →   double

* 2019-01-01  →  date

* "hello world"  →  text/keywod

**查看 mapping**：`GET /index/_mapping/`

### 10.8 手动管理mapping

**查询所有索引的映射**：`GET /_mapping`

#### 10.8.1 重要：创建映射

创建索引后，应该立即手动创建映射

```json
// 先创建索引
PUT book

# 建立映射
PUT book/_mapping
{
    "properties": {
        "name": {
            "type": "text"
        },
        "description": {
            "type": "text",               # text类型
            "analyzer":"english",		  # 指定索引时的分词器
            "search_analyzer":"english"   # 指定搜索时的分词器
        },
        "pic":{
            "type":"text",
            "index":false   # 是否走索引
        },
        "studymodel":{
            "type":"text"
        }
    }
}
```

##### Text 文本类型

1. analyzer：

   通过 analyzer 属性指定分词器。

   上边指定了 analyzer 是指在索引和搜索都使用 english，如果单独想定义搜索时使用的分词器则可以通过 search_analyzer 属性。

2. index

   index属性指定是否索引。

   默认为 `index=true`，即要进行索引，**只有进行索引才可以从索引库搜索到**。

   但是也有一些内容不需要索引，比如：商品图片地址只被用来展示图片，不进行搜索图片，此时可以将 index 设置为 false。

   删除索引，重新创建映射，将 pic 的 index 设置为 false，尝试根据 pic 去搜索，结果搜索不到数据。

3. store

   是否在 source 之外存储，每个文档索引后会在 ES 中保存一份原始文档，存放在`_source`中，一般情况下不需要设置 store 为 true，因为在 `_source` 中已经有一份原始文档了

**测试**：

* 插入文档

```json
PUT /book/_doc/1
{
    "name":"Bootstrap开发框架",
    "description":"Bootstrap是由Twitter推出的一个前台页面开发框架，在行业之中使用较为广泛。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长页面开发的程序人员）轻松的实现一个不受浏览器限制的精美界面效果。",
    "pic":"group1/M00/00/01/wKhlQFqO4MmAOP53AAAcwDwm6SU490.jpg",
    "studymodel":"201002"
}
```

* 查询：

```json
GET /book/_search?q=name:开发
GET /book/_search?q=description:开发
GET /book/_search?q=pic:group1/M00/00/01/wKhlQFqO4MmAOP53AAAcwDwm6SU490.jpg
GET /book/_search?q=studymodel:201002
```

通过测试发现：name 和 description 都支持全文检索，pic不可作为查询条件。

##### keyword 关键字字段

目前已经取代了 `"index": false`。

上边介绍的 text 文本字段在映射时要设置分词器，keyword 字段为关键字字段，通常搜索 keyword 是按照整体搜索，所以创建keyword 字段的索引时是不进行分词的，比如：邮政编码、手机号码、身份证等，keyword 字段通常用于过虑、排序、聚合等。

```json
"pic":{
	"type":"keyword"
},
```

##### date 日期类型

日期类型不用设置分词器。

通常日期类型的字段用于排序。

通过 format 设置日期格式

例子：下边的设置允许 date 字段存储年月日时分秒、年月日及毫秒三种格式。

```json
PUT book/_mapping
{
    "properties": {
        "timestamp": {
            "type":   "date",
            "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"
        }
    }
}
```

* 插入文档：

```json
POST book/_doc/3 
{
    "name": "spring开发基础",
    "description": "spring 在java领域非常流行，java程序员都在用。",
    "studymodel": "201001",
    "pic":"group1/M00/00/01/wKhlQFqO4MmAOP53AAAcwDwm6SU490.jpg",
    "timestamp":"2018-07-04 18:28:58"
}
```

* 查看结果

```json
GET book/_doc/3
{
    "_index" : "book",
    "_type" : "_doc",
    "_id" : "3",
    "_version" : 1,
    "_seq_no" : 1,
    "_primary_term" : 1,
    "found" : true,
    "_source" : {
        "name" : "spring开发基础",
        "description" : "spring 在java领域非常流行，java程序员都在用。",
        "studymodel" : "201001",
        "pic" : "group1/M00/00/01/wKhlQFqO4MmAOP53AAAcwDwm6SU490.jpg",
        "timestamp" : "2018-07-04 18:28:58"
    }
}
```

##### 数值类型

下边是ES支持的数值类型

![ES支持的数值类型](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261416144.png)

1. 尽量选择范围小的类型，提高搜索效率

2. 对于浮点数尽量用比例因子，比如一个价格字段，单位为元，我们将比例因子设置为 100 这在 ES 中会按分存储，映射如下：

   ```json
   "price": {
       "type": "scaled_float",
       "scaling_factor": 100
   },
   ```

   由于比例因子为 100，如果我们输入的价格是 23.45 则ES中会将 23.45 乘以 100 存储在 ES 中。

   如果输入的价格是 23.456，ES 会将 23.456 乘以 100 再取一个接近原始值的数，得出 2346。

   使用比例因子的好处是整型比浮点型更易压缩，节省磁盘空间。

   如果比例因子不适合，则从下表选择范围小的去用：

   更新已有映射，并插入文档：

   ```json
   PUT book/doc/3
   {
       "name": "spring开发基础",
       "description": "spring 在java领域非常流行，java程序员都在用。",
       "studymodel": "201001",
       "pic":"group1/M00/00/01/wKhlQFqO4MmAOP53AAAcwDwm6SU490.jpg",
       "timestamp":"2018-07-04 18:28:58",
       "price":38.6
   }
   ```

#### 10.8.2 修改映射

只能创建 index 时手动建立 mapping，或者新增 field mapping，**但是不能 update field mapping**。

因为已有数据按照映射早已分词存储好。如果修改，那这些存量数据怎么办。

新增一个字段mapping

```json
PUT /book/_mapping/
{
    "properties" : {
        "new_field" : {
            "type" : "text",
            "index": "false"
        }
    }
}
```

如果修改mapping，会报错

```json
PUT /book/_mapping/
{
    "properties" : {
        "studymodel" : {
            "type" :    "keyword"
        }
    }
}

// 返回：
{
    "error": {
        "root_cause": [
            {
                "type": "illegal_argument_exception",
                "reason": "mapper [studymodel] of different type, current_type [text], merged_type [keyword]"
            }
        ],
        "type": "illegal_argument_exception",
        "reason": "mapper [studymodel] of different type, current_type [text], merged_type [keyword]"
    },
    "status": 400
}
```

#### 10.8.4 删除映射

通过删除索引来删除映射。

### 10.9 复杂数据类型

#### 10.9.1 multivalue field

数组类型：`{ "tags": [ "tag1", "tag2"]}`

建立索引时与 string 是一样的，数据类型不能混，即数据类型需要一致

#### 10.9.2 empty field

空值：`null`

空数组：`[]`，`[null]`

#### 10.9.3 object field

```json
PUT /company/_doc/1
{
    "address": {
        "country": "china",
        "province": "guangdong",
        "city": "guangzhou"
    },
    "name": "jack",
    "age": 27,
    "join_date": "2019-01-01"
}
```

其中 address：object 类型

查询company的映射：

```json
GET /company/_mapping
{
    "company" : {  // index
        "mappings" : {  // mapping
            "properties" : {  // 属性
                "address" : {  // address，object类型
                    "properties" : {
                        "city" : {  // address.city
                            "type" : "text",
                            "fields" : {
                                "keyword" : {
                                    "type" : "keyword",
                                    "ignore_above" : 256
                                }
                            }
                        },
                        "country" : {  // address.country
                            "type" : "text",
                            "fields" : {
                                "keyword" : {
                                    "type" : "keyword",
                                    "ignore_above" : 256
                                }
                            }
                        },
                        "province" : {  // address.province
                            "type" : "text",
                            "fields" : {
                                "keyword" : {
                                    "type" : "keyword",
                                    "ignore_above" : 256
                                }
                            }
                        }
                    }
                },
                "age" : {
                    "type" : "long"
                },
                "join_date" : {
                    "type" : "date"
                },
                "name" : {
                    "type" : "text",
                    "fields" : {
                        "keyword" : {
                            "type" : "keyword",
                            "ignore_above" : 256
                        }
                    }
                }
            }
        }
    }
}
```

对象：

```json
{
    "address": {
        "country": "china",
        "province": "guangdong",
        "city": "guangzhou"
    },
    "name": "jack",
    "age": 27,
    "join_date": "2017-01-01"
}
```

底层存储格式：

```json
{
    "name": [jack],
    "age": [27],
    "join_date": [2017-01-01],
    "address.country": [china],
    "address.province": [guangdong],
    "address.city": [guangzhou]
}
```

对象数组：

```json
{
    "authors": [
        { "age": 26, "name": "Jack White"},
        { "age": 55, "name": "Tom Jones"},
        { "age": 39, "name": "Kitty Smith"}
    ]
}
```

存储格式：

```json
{
    "authors.age": [26, 55, 39],
    "authors.name": [jack, white, tom, jones, kitty, smith]
}
```

