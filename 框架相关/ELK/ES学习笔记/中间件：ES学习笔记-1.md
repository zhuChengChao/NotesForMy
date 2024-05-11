# 中间件：ES学习笔记-1

> 最近有点喜欢对照着学习...:no_mouth:，就很顺带的又看了一波课程：[尚硅谷：ElasticSearch教程入门到精通](https://www.bilibili.com/video/BV1hh411D7sb)，就当查漏补缺了呗，课程提供了 PDF 课件，看着总不是很顺眼，就给它转成 markdown 的了，总的拆分了2篇，此为1/2篇。

## 1. Elasticsearch概述

### 1.1 概述

![ES简介2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261419003.png)

The Elastic Stack，包括 Elasticsearch、Kibana、Beats 和 Logstash（也称为 ELK Stack）。能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。 Elaticsearch，简称为 ES， ES 是一个开源的高扩展的分布式全文搜索引擎， 是整个 Elastic Stack 技术栈的核心。它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理 PB 级别的数据。  

### 1.2 全文搜索引擎

Google，百度类的网站搜索，它们都是根据网页中的关键字生成索引，我们在搜索的时候输入关键字，它们会将该关键字即索引匹配到的所有网页返回；还有常见的项目中应用日志的搜索等等。对于这些非结构化的数据文本，关系型数据库搜索不是能很好的支持。

一般传统数据库，全文检索都实现的很鸡肋，因为一般也没人用数据库存文本字段。进行全文检索需要扫描整个表，如果数据量大的话即使对 SQL 的语法优化，也收效甚微。建立了索引，但是维护起来也很麻烦，对于 insert 和 update 操作都会重新构建索引。

基于以上原因可以分析得出，在一些生产环境中，使用常规的搜索方式，性能是非常差的：

* 搜索的数据对象是大量的非结构化的文本数据。
* 文件记录量达到数十万或数百万个甚至更多。
* 支持大量基于交互式文本的查询。
* 需求非常灵活的全文搜索查询。
* 对高度相关的搜索结果的有特殊需求，但是没有可用的关系数据库可以满足。  
* 对不同记录类型、非文本数据操作或安全事务处理的需求相对较少的情况。

为了解决结构化数据搜索和非结构化数据搜索性能问题，我们就需要专业，健壮，强大的全文搜索引擎。

这里说到的全文搜索引擎指的是目前广泛应用的主流搜索引擎。它的工作原理是计算机索引程序通过扫描文章中的每一个词，对每一个词建立一个索引，指明该词在文章中出现的次数和位置，当用户查询时，检索程序就根据事先建立的索引进行查找，并将查找的结果反馈给用户的检索方式。这个过程类似于通过字典中的检索字表查字的过程。  

### 1.3 Elasticsearch & Solr

Lucene 是 Apache 软件基金会 Jakarta 项目组的一个子项目，提供了一个简单却强大的应用程式接口，能够做全文索引和搜寻。在 Java 开发环境里 Lucene 是一个成熟的免费开源工具。就其本身而言， Lucene 是当前以及最近几年最受欢迎的免费 Java 信息检索程序库。

但 Lucene 只是一个提供全文搜索功能类库的核心工具包，而真正使用它还需要一个完善的服务框架搭建起来进行应用。

目前市面上流行的搜索引擎软件，主流的就两款： Elasticsearch 和 Solr，这两款都是基于 Lucene 搭建的，可以独立部署启动的搜索引擎服务软件。由于内核相同，所以两者除了服务器安装、部署、管理、集群以外，对于数据的操作修改、添加、保存、查询等等都十分类似。

在使用过程中，一般都会将 Elasticsearch 和 Solr 这两个软件对比，然后进行选型。这两个搜索引擎都是流行的，先进的开源搜索引擎。它们都是围绕核心底层搜索库 Lucene 构建的，但它们又是不同的。像所有东西一样，每个都有其优点和缺点。

Elasticsearch 和 Solr 都是开源搜索引擎，那么我们在使用时该如何选择呢？

* Google 搜索趋势结果表明，与 Solr 相比， Elasticsearch 具有很大的吸引力，但这并不意味着 Apache Solr 已经死亡。虽然有些人可能不这么认为，但 Solr 仍然是最受欢迎的搜索引擎之一，拥有强大的社区和开源支持。
* 与 Solr 相比， Elasticsearch 易于安装且非常轻巧。此外，你可以在几分钟内安装并运行 Elasticsearch。但是，如果 Elasticsearch 管理不当，这种易于部署和使用可能会成为一个问题。基于 JSON 的配置很简单，但如果要为文件中的每个配置指定注释，那么它不适合您。总的来说，如果你的应用使用的是 JSON，那么 Elasticsearch 是一个更好的选择。否则，请使用 Solr，因为它的 schema.xml 和 solrconfig.xml 都有很好的文档记录。
* Solr 拥有更大，更成熟的用户，开发者和贡献者社区。 ES 虽拥有的规模较小但活跃的用户社区以及不断增长的贡献者社区。Solr 贡献者和提交者来自许多不同的组织，而 Elasticsearch 提交者来自单个公司。
* Solr 更成熟，但 ES 增长迅速，更稳定。
* Solr 是一个非常有据可查的产品，具有清晰的示例和 API 用例场景。 Elasticsearch 的文档组织良好，但它缺乏好的示例和清晰的配置说明。  

**那么，到底是 Solr 还是 Elasticsearch？**  

有时很难找到明确的答案。无论您选择 Solr 还是 Elasticsearch，首先需要了解正确的用例和未来需求。总结他们的每个属性。

* 由于易于使用， Elasticsearch 在新开发者中更受欢迎。一个下载和一个命令就可以启动一切。
* 如果除了搜索文本之外还需要它来处理分析查询， Elasticsearch 是更好的选择
* 如果需要分布式索引，则需要选择 Elasticsearch。对于需要良好可伸缩性和以及性能分布式环境， Elasticsearch 是更好的选择。
* Elasticsearch 在开源日志管理用例中占据主导地位，许多组织在 Elasticsearch 中索引它们的日志以使其可搜索。
* 如果你喜欢监控和指标，那么请使用 Elasticsearch，因为相对于 Solr， Elasticsearch 暴露了更多的关键指标  

### 1.4 Elasticsearch应用案例

* GitHub: 2013 年初，抛弃了 Solr，采取 Elasticsearch 来做 PB 级的搜索。 “GitHub 使用 Elasticsearch 搜索 20TB 的数据，包括 13 亿文件和 1300 亿行代码”。
* 维基百科：启动以 Elasticsearch 为基础的核心搜索架构
* SoundCloud： “SoundCloud 使用 Elasticsearch 为 1.8 亿用户提供即时而精准的音乐搜索服务”。
* 百度：目前广泛使用 Elasticsearch 作为文本数据分析，采集百度所有服务器上的各类指标数据及用户自定义数据，通过对各种数据进行多维分析展示，辅助定位分析实例异常或业务层面异常。目前覆盖百度内部 20 多个业务线（包括云分析、网盟、预测、文库、直达号、钱包、 风控等），单集群最大 100 台机器， 200 个 ES 节点，每天导入 30TB+数据。
* 新浪：使用 Elasticsearch 分析处理 32 亿条实时日志。
* 阿里：使用 Elasticsearch 构建日志采集和分析体系。
* Stack Overflow：解决 Bug 问题的网站，全英文，编程人员交流的网站。

## 2. Elasticsearch入门  

### 2.1 ES安装

> 下载、解压、运行即可

### 2.2 ES基本操作

#### 2.2.1 RESTful

REST 指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful。 Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在请求之间是无状态的。从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器回答，这十分适合云计算之类的环境。客户端可以缓存数据以改进性能。

在服务器端，应用程序状态和功能可以分为各种资源。资源是一个有趣的概念实体，它向客户端公开。资源的例子有：应用程序对象、数据库记录、算法等等。每个资源都使用 URI(Universal Resource Identifier) 得到一个唯一的地址。所有资源都共享统一的接口，以便在客户端和服务器之间传输状态。使用的是标准的 HTTP 方法，比如 GET、 PUT、 POST 和 DELETE。

在 REST 样式的 Web 服务中，每个资源都有一个地址。资源本身都是方法调用的目标，方法列表对所有资源都是一样的。这些方法都是标准方法，包括 HTTP GET、 POST、PUT、 DELETE，还可能包括 HEAD 和 OPTIONS。**简单的理解就是，如果想要访问互联网上的资源，就必须向资源所在的服务器发出请求，请求体中必须包含资源的网络路径， 以及对资源进行的操作(增删改查)。**  

#### 2.2.2 数据格式

Elasticsearch 是面向文档型数据库，**一条数据在这里就是一个文档**。这里将 Elasticsearch 里存储文档数据和关系型数据库 MySQL 存储数据的概念进行一个类比：

![数据库和ES类比](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261419143.png)

ES 里的 Index 可以看做一个库，而 Types 相当于表， Documents 则相当于表的行。

这里 Types 的概念已经被逐渐弱化， Elasticsearch 6.X 中，一个 index 下已经只能包含一个 type，Elasticsearch 7.X 中，Type 的概念已经被删除了。  

ES 中用 JSON 作为文档序列化的格式，比如一条用户信息：  

```json
{
    "name" : "John",
    "sex" : "Male",
    "age" : 25,
    "birthDate": "1990/05/01",
    "about" : "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

#### 2.2.3 HTTP 操作之索引操作

##### **1) 创建索引**

对比关系型数据库，创建索引就等同于创建数据库，在 Postman 中，向 ES 服务器发PUT请求 ： http://127.0.0.1:9200/shopping

请求后，服务器返回响应：

```json
{
    "acknowledged": true,  // 响应结果，true 操作成功
    "shards_acknowledged": true,  // 分片结果，分片操作成功
    "index": "shopping"  // 索引名称
}
```

> 注意：创建索引库的分片数默认 1 片，在 7.0.0 之前的 Elasticsearch 版本中，默认 5 片  

如果重复添加索引，会返回错误信息

```json
{
    "error": {
        "root_cause": [
            {
                "type": "resource_already_exists_exception",
                "reason": "index [shopping/YlthILbeS6Wah9YlaIIvvA] already exists",
                "index_uuid": "YlthILbeS6Wah9YlaIIvvA",
                "index": "shopping"
            }
        ],
        "type": "resource_already_exists_exception",
        "reason": "index [shopping/YlthILbeS6Wah9YlaIIvvA] already exists",
        "index_uuid": "YlthILbeS6Wah9YlaIIvvA",
        "index": "shopping"
    },
    "status": 400
}
```

##### **2) 查看所有索引**

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/_cat/indices?v

这里请求路径中的 `_cat` 表示查看的意思， indices 表示索引，所以整体含义就是查看当前 ES 服务器中的所有索引，就好像 MySQL 中的 show tables 的感觉，服务器响应结果如下：

![查看所有索引](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261419697.png)

| 表头           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| health         | 当前服务器健康状态：green(集群完整)、yellow(单点正常/集群不完整)、red(单点不正常) |
| status         | 索引打开、关闭状态                                           |
| index          | 索引名                                                       |
| uuid           | 索引统一编号                                                 |
| pri            | 主分片数量                                                   |
| rep            | 副本数量                                                     |
| docs.count     | 可用文档数量                                                 |
| docs.deleted   | 文档删除状态（逻辑删除）                                     |
| store.size     | 主分片和副分片整体占空间大小                                 |
| pri.store.size | 主分片占空间大小                                             |

##### **3）查看单个索引**

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/shopping

查看索引向 ES 服务器发送的请求路径和创建索引是一致的。但是 HTTP 方法不一致。这里可以体会一下 RESTful 的意义，请求后，服务器响应结果如下：

```json
{
    "shopping": {  // 索引名
        "aliases": {},  // 别名
        "mappings": {},  // 映射
        "settings": {  // 设置
            "index": {  // 设置-索引
                "creation_date": "1638193264926",  // 设置-索引-创建时间
                "number_of_shards": "1",  // 设置-索引-主分片数量
                "number_of_replicas": "1",  // 设置-索引-副的分片数量
                "uuid": "YlthILbeS6Wah9YlaIIvvA",  // 设置-索引-唯一标识
                "version": {  // 设置-索引-版本
                    "created": "7080099"
                },
                "provided_name": "shopping"  // 设置-索引-名称
            }
        }
    }
}
```

##### **4) 删除索引**

在 Postman 中，向 ES 服务器发 DELETE 请求 ： http://127.0.0.1:9200/shopping

响应：`"acknowledged": true`

重新访问索引时，服务器返回响应： 索引不存在

```json
{
    "error": {
        "root_cause": [
            {
                "type": "index_not_found_exception",
                "reason": "no such index [shopping]",
                "resource.type": "index_or_alias",
                "resource.id": "shopping",
                "index_uuid": "_na_",
                "index": "shopping"
            }
        ],
        "type": "index_not_found_exception",
        "reason": "no such index [shopping]",
        "resource.type": "index_or_alias",
        "resource.id": "shopping",
        "index_uuid": "_na_",
        "index": "shopping"
    },
    "status": 404
}
```

#### 2.2.4 HTTP 操作之文档操作

##### **1) 创建文档**

索引已经创建好了，接下来我们来创建文档，并添加数据。这里的文档可以类比为关系型数据库中的表数据，添加的数据格式为 JSON 格式在 Postman 中，向 ES 服务器发 POST 请求 ： http://127.0.0.1:9200/shopping/_doc，请求体内容为：  

```json
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```

此处发送请求的方式必须为 POST，不能是 PUT，否则会发生错误，服务器响应结果如下：

```json
{
    "_index": "shopping",  // 索引
    "_type": "_doc",  // 类型-文档
    "_id": "2OX2a30BHOAbRDQfU2qI",  // 唯一标识，可以类比为 MySQL 中的主键，随机生成
    "_version": 1,  // 版本
    "result": "created",  // 结果，这里的 create 表示创建成功
    "_shards": {
        "total": 2,  // 分片：总数
        "successful": 1,  // 分片：成功
        "failed": 0  // 分片：失败
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

上面的数据创建后，由于没有指定数据唯一性标识（ID），默认情况下， ES 服务器会随机生成一个。

如果想要自定义唯一性标识，需要在创建时指定： http://127.0.0.1:9200/shopping/_doc/1，结果如下：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```

> 此处需要注意：如果增加数据时明确数据主键，那么请求方式也可以为 PUT 

##### **2) 查看文档**

查看文档时，需要指明文档的唯一性标识，类似于 MySQL 中数据的主键查询

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/shopping/_doc/1，查询成功后，服务器响应结果：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "_seq_no": 1,
    "_primary_term": 1,
    "found": true,  // 查询结果，true 表示查找到， false 表示未查找到
    "_source": {  // 文档源信息
        "title": "小米手机",
        "category": "小米",
        "images": "http://www.gulixueyuan.com/xm.jpg",
        "price": 3999.00
    }
}
```

##### **3) 修改文档**

和新增文档一样，输入相同的 URL 地址请求，如果请求体变化，会将原有的数据内容覆盖在 Postman 中，向 ES 服务器发 POST 请求 ： http://127.0.0.1:9200/shopping/_doc/1，请求体内容为:  

```json
{
    "title":"华为手机",
    "category":"华为",
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":4999.00
}
```

修改成功后，服务器响应结果：  

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 2,  // 版本号+1
    "result": "updated",  // updated 表示数据被更新
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```

##### **4) 修改字段**

修改数据时，也可以只修改某一给条数据的局部信息 在 Postman 中，向 ES 服务器发 POST 请求 ： http://127.0.0.1:9200/shopping/_update/1，请求体内容为：  

```json
{
    "doc": {
        "price":3000.00
    }
}
```

修改成功后，服务器响应结果：  

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 3,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```

根据唯一性标识，查询文档数据，文档数据已经更新，发送 GET 请求：http://127.0.0.1:9200/shopping/_doc/1

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 3,
    "_seq_no": 3,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "title":"华为手机",
        "category":"华为",
        "images":"http://www.gulixueyuan.com/hw.jpg",
        "price":3000.00
    }
}
```

##### **5) 删除文档**

删除一个文档不会立即从磁盘上移除，它只是被标记成已删除（逻辑删除）。

在 Postman 中，向 ES 服务器发 DELETE 请求 ： http://127.0.0.1:9200/shopping/_doc/1，删除成功，服务器响应结果：  

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 4,  // 对数据的操作，都会更新版本
    "result": "deleted", // deleted 表示数据被标记为删除
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 4,
    "_primary_term": 1
}
```

删除后再查询当前文档信息，发送 GET 请求：http://127.0.0.1:9200/shopping/_doc/1，相应如下：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "found": false
}
```

如果删除一个并不存在的文档：重复发送上述请求，相应如下：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "result": "not_found",  // 未查询到
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 5,
    "_primary_term": 1
}
```

##### **6) 条件删除文档**

一般删除数据都是根据文档的唯一性标识进行删除，实际操作时，也可以根据条件对多条数据进行删除

首先分别增加多条数据：

```json
POST请求：http://127.0.0.1:9200/shopping/_doc/1
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":4000.00
} 

POST请求：http://127.0.0.1:9200/shopping/_doc/2
{
    "title":"华为手机",
    "category":"华为",
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":4000.00
}
```

向 ES 服务器发 POST 请求 ： http://127.0.0.1:9200/shopping/_delete_by_query，请求体内容为：  

```json
{
    "query":{
        "match":{
            "price":4000.00
        }
    }
}
```

删除成功后，服务器响应结果：

```json
{
    "took": 575,  // 耗时
    "timed_out": false,  // 是否耗时
    "total": 2,  // 总的数量
    "deleted": 2,  // 删除数量
    "batches": 1,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1.0,
    "throttled_until_millis": 0,
    "failures": []
}
```

#### 2.2.4 HTTP 操作之映射操作

有了索引库，等于有了数据库中的 database。

接下来就需要建索引库(index)中的映射了，类似于数据库(database)中的表结构(table)。创建数据库表需要设置字段名称，类型，长度，约束等；索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射(mapping)。  

##### **1）创建映射**

在 Postman 中，向 ES 服务器发 PUT 请求 ： http://127.0.0.1:9200/student/_mapping，请求体内容为  

```json
{
    "properties": {
        "name":{  // 字段名
            "type": "text",  // 类型
            "index": true    // 是否索引
        },
        "sex":{
            "type": "text",
            "index": false
        },
        "age":{
            "type": "long",
            "index": false
        }
    }
}
```

服务器响应结果如下：  

```json
{
    "acknowledged": true
}
```

映射数据说明：

* 字段名：任意填写，下面指定许多属性，例如： title、 subtitle、 images、 price

* type：类型， Elasticsearch 中支持的数据类型非常丰富，说几个关键的：

  * String 类型，又分两种：

    * text：可分词  
    * keyword：不可分词，数据会作为完整字段进行匹配

  * Numerical：数值类型，分两类

    基本数据类型： long、 integer、 short、 byte、 double、 float、 half_float

    浮点数的高精度类型： scaled_float

  * Date：日期类型

  * Array：数组类型

  * Object：对象

* index：是否索引，默认为 true，也就是说你不进行任何配置，所有字段都会被索引。

  * true：字段会被索引，则可以用来进行搜索
  * false：字段不会被索引，不能用来搜索

* store：是否将数据进行独立存储，默认为 false

  原始的文本会存储在 `_source` 里面，默认情况下其他提取出来的字段都不是独立存储的，是从 `_source` 里面提取出来的。当然你也可以独立的存储某个字段，只要设置 `"store": true` 即可，获取独立存储的字段要比从 `_source` 中解析快得多，但是也会占用更多的空间，所以要根据实际业务需求来设置。

* analyzer：分词器，这里的 ik_max_word 即使用 ik 分词器

##### **2) 查看映射**

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_mapping  

服务器响应结果如下：

```json
{
    "student": {
        "mappings": {
            "properties": {
                "age": {
                    "type": "long",
                    "index": false
                },
                "name": {
                    "type": "text"
                },
                "sex": {
                    "type": "text",
                    "index": false
                }
            }
        }
    }
}
```

##### **3) 索引映射关联**

在 Postman 中，向 ES 服务器发 PUT 请求 ： http://127.0.0.1:9200/student1

```json
{
    "settings": {},
    "mappings": {
        "properties": {
            "name":{
                "type": "text",
                "index": true
            },
            "sex":{
                "type": "text",
                "index": false
            },
            "age":{
                "type": "long",
                "index": false
            }
        }
    }
}
```

服务器响应结果如下：  

```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "student1"
}
```

#### 2.2.4 HTTP 操作之高级查询

Elasticsearch 提供了基于 JSON 提供完整的查询 DSL 来定义查询

定义数据 :  

```json
# POST /student/_doc/1001
{
    "name":"zhangsan",
    "nickname":"zhangsan",
    "sex":"男",
    "age":30
}
# POST /student/_doc/1002
{
    "name":"lisi",
    "nickname":"lisi",
    "sex":"男",
    "age":20
}
# POST /student/_doc/1003
{
    "name":"wangwu",
    "nickname":"wangwu",
    "sex":"女",
    "age":40
}
# POST /student/_doc/1004
{
    "name":"zhangsan1",
    "nickname":"zhangsan1",
    "sex":"女",
    "age":50
}
# POST /student/_doc/1005
{
    "name":"zhangsan2",
    "nickname":"zhangsan2",
    "sex":"女",
    "age":30
}
```

##### **1) 查询所有文档**

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "match_all": {}
    }
}
# "query"：这里的 query 代表一个查询对象，里面可以有不同的查询属性
# "match_all"：查询类型，例如： match_all(代表查询所有)， match， term ， range 等等
# {查询条件}：查询条件会根据类型的不同，写法也有差异
```

服务器响应结果如下：

```json
{
    "took": 1,  // 查询花费时间，单位毫秒
    "timed_out": false,  // 是否超
    "_shards": {  // 分片信息
        "total": 1,  // 总数
        "successful": 1, // 成功
        "skipped": 0,  // 忽略
        "failed": 0  // 失败
    },
    "hits": {  // 搜索命中结果
        "total": {  // 搜索条件匹配的文档总数
            "value": 5,
            "relation": "eq"  // 计数规则,eq 表示计数准确,gte 表示计数不准确
        },
        "max_score": 1.0,  // 匹配度分值
        "hits": [  // 命中结果集合
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "1001",
                "_score": 1.0,
                "_source": {
                    "name": "zhangsan",
                    "nickname": "zhangsan",
                    "sex": "男",
                    "age": 30
                }
            },
            // ...
        ]
    }
}
```

##### **2) 匹配查询**

match 匹配类型查询，会把查询条件进行分词，然后进行查询，多个词条之间是 or 的关系，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "match": {
            "name":"zhangsan"
        }
    }
}
```

服务器响应结果为：

```json
{
	"took": 0,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 1,
			"relation": "eq"
		},
		"max_score": 1.3862942,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": 1.3862942,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			}
		}]
	}
}
```

##### **3) 字段匹配查询**

multi_match 与 match 类似，不同的是它可以在多个字段中查询。

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "multi_match": {
            "query": "zhangsan",
            "fields": ["name","nickname"]
        }
    }
}
```

服务器响应结果：

```json
{
	"took": 1,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 1,
			"relation": "eq"
		},
		"max_score": 1.3862942,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": 1.3862942,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			}
		}]
	}
}
```

##### **4) 关键字精确查询**

term 查询，精确的关键词匹配查询，不对查询条件进行分词。

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "term": {
            "name": {
                "value": "zhangsan"
            }
        }
    }
}
```

服务器响应结果：

```json
{
	"took": 0,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 1,
			"relation": "eq"
		},
		"max_score": 1.3862942,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": 1.3862942,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			}
		}]
	}
}
```

##### **5) 多关键字精确查询**

terms 查询和 term 查询一样，但它允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件，类似于 mysql 的 in。

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "terms": {
            "name": ["zhangsan","lisi"]
        }
    }
}
```

服务器响应结果：  

```json
{
	"took": 2,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 2,
			"relation": "eq"
		},
		"max_score": 1.0,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": 1.0,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			}
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1002",
			"_score": 1.0,
			"_source": {
				"name": "lisi",
				"nickname": "lisi",
				"sex": "男",
				"age": 20
			}
		}]
	}
}
```

##### **6) 指定查询字段**

默认情况下， Elasticsearch 在搜索的结果中，会把文档中保存在 `_source` 的所有字段都返回。如果我们只想获取其中的部分字段，我们可以添加 `_source` 的过滤

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "_source": ["name","nickname"],
    "query": {
        "terms": {
            "nickname": ["zhangsan"]
        }
    }
}
```

服务器响应结果：

```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "1001",
                "_score": 1.0,
                "_source": {
                    "name": "zhangsan",
                    "nickname": "zhangsan"
                }
            }
        ]
    }
}
```

##### **7) 过滤字段**

我们也可以通过：  

* includes：来指定想要显示的字段
* excludes：来指定不想要显示的字段

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "_source": {
        "includes": ["name","nickname"]
    },
    "query": {
        "terms": {
            "nickname": ["zhangsan"]
        }
    }
}
```

服务器响应结果：

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "1001",
                "_score": 1.0,
                "_source": {
                    "name": "zhangsan",
                    "nickname": "zhangsan"
                }
            }
        ]
    }
}
```

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "_source": {
        "excludes": ["name","nickname"]
    },
    "query": {
        "terms": {
            "nickname": ["zhangsan"]
        }
    }
}
```

服务器响应结果：

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "1001",
                "_score": 1.0,
                "_source": {
                    "sex": "男",
                    "age": 30
                }
            }
        ]
    }
}
```

##### **8) 组合查询**

> 这里需要注意的是创建的 student 索引，对于 age 和 sex 是没有创建索引的，即index=false，删了之后再重建一下吧...

`bool`把各种其它查询通过`must`（必须）、 `must_not`（必须不）、 `should`（应该）的方式进行组合

在 Postman 中，向 ES 服务器发 GET 请求 ：http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "bool": {
            "must": [  // 必须:名字中必须有zhangsan
                {
                    "match": {
                        "name": "zhangsan"
                    }
                }
            ],
            "must_not": [  // 必须不:年龄不能是40
                {
                    "match": {
                        "age": "40"
                    }
                }
            ],
            "should": [  // 应该:性别应该是男
                {
                    "match": {
                        "sex": "男"
                    }
                }
            ]
        }
    }
}
```

服务器响应结果：

```json
{
    "took": 973,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 2.261763,
        "hits": [
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "1001",
                "_score": 2.261763,
                "_source": {
                    "name": "zhangsan",
                    "nickname": "zhangsan",
                    "sex": "男",
                    "age": 30
                }
            }
        ]
    }
}
```

##### **9) 范围查询**

range 查询找出那些落在指定区间内的数字或者时间，range 查询允许以下字符  

| 操作符 | 说明          |
| ------ | ------------- |
| gt     | 大于 `>`      |
| gte    | 大于等于 `>=` |
| lt     | 小于 `<`      |
| lte    | 小于等于 `<=` |

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "range": {
            "age": {
                "gte": 30,
                "lte": 35
            }
        }
    }
}
```

服务器响应结果：  

```json
{
	"took": 0,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 2,
			"relation": "eq"
		},
		"max_score": 1.0,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": 1.0,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			}
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1005",
			"_score": 1.0,
			"_source": {
				"name": "zhangsan2",
				"nickname": "zhangsan2",
				"sex": "女",
				"age": 30
			}
		}]
	}
}
```

##### **10) 模糊查询**

返回包含与搜索字词相似的字词的文档。

编辑距离是将一个术语转换为另一个术语所需的一个字符更改的次数，这些更改可以包括：

* 更改字符（box → fox）
* 删除字符（black → lack）  
* 插入字符（sic → sick）
* 转置两个相邻字符（act → cat）

为了找到相似的术语， fuzzy 查询会在指定的编辑距离内创建一组搜索词的所有可能的变体或扩展。然后查询返回每个扩展的完全匹配。

通过 fuzziness 修改编辑距离。一般使用默认值 AUTO，根据术语的长度生成编辑距离。

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "fuzzy": {
            "name": {
                "value": "zhangsan"
            }
        }
    }
}
```

服务器响应结果：  

```json
{
	"took": 3,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 3,
			"relation": "eq"
		},
		"max_score": 1.3862942,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": 1.3862942,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			}
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1004",
			"_score": 1.2130076,
			"_source": {
				"name": "zhangsan1",
				"nickname": "zhangsan1",
				"sex": "女",
				"age": 50
			}
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1005",
			"_score": 1.2130076,
			"_source": {
				"name": "zhangsan2",
				"nickname": "zhangsan2",
				"sex": "女",
				"age": 30
			}
		}]
	}
}
```

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "fuzzy": {
            "name": {
                "value": "zhangsan",
                "fuzziness": 2
            }
        }
    }
}
```

服务器响应结果：  

```json
{
	"took": 2,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 3,
			"relation": "eq"
		},
		"max_score": 1.3862942,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": 1.3862942,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			}
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1004",
			"_score": 1.2130076,
			"_source": {
				"name": "zhangsan1",
				"nickname": "zhangsan1",
				"sex": "女",
				"age": 50
			}
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1005",
			"_score": 1.2130076,
			"_source": {
				"name": "zhangsan2",
				"nickname": "zhangsan2",
				"sex": "女",
				"age": 30
			}
		}]
	}
}
```

##### **11) 单字段排序**

sort 可以让我们按照不同的字段进行排序，并且通过 order 指定排序的方式。 desc 降序， asc 升序。

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
	"query": {
        "match_all": {}
    },
    "sort": [{
        "age": {
            "order":"desc"
        }
    }]
}
```

服务器响应结果：

```json
{
	"took": 2,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 5,
			"relation": "eq"
		},
		"max_score": null,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1004",
			"_score": null,
			"_source": {
				"name": "zhangsan1",
				"nickname": "zhangsan1",
				"sex": "女",
				"age": 50
			},
			"sort": [50]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1003",
			"_score": null,
			"_source": {
				"name": "wangwu",
				"nickname": "wangwu",
				"sex": "女",
				"age": 40
			},
			"sort": [40]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": null,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			},
			"sort": [30]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1005",
			"_score": null,
			"_source": {
				"name": "zhangsan2",
				"nickname": "zhangsan2",
				"sex": "女",
				"age": 30
			},
			"sort": [30]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1002",
			"_score": null,
			"_source": {
				"name": "lisi",
				"nickname": "lisi",
				"sex": "男",
				"age": 20
			},
			"sort": [20]
		}]
	}
}
```

##### **12) 多字段排序**

假定我们想要结合使用 age 和 `_score` 进行查询，并且匹配的结果首先按照年龄排序，然后按照相关性得分排序。

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search  

```json
{
    "query": {
        "match_all": {}
    },
    "sort": [
        {
            "age": {
                "order": "desc"
            }
        },
        {
            "_score":{
                "order": "desc"
            }
        }
    ]
}
```

服务器响应结果： 

```json
{
	"took": 1,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 5,
			"relation": "eq"
		},
		"max_score": null,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1004",
			"_score": 1.0,
			"_source": {
				"name": "zhangsan1",
				"nickname": "zhangsan1",
				"sex": "女",
				"age": 50
			},
			"sort": [50, 1.0]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1003",
			"_score": 1.0,
			"_source": {
				"name": "wangwu",
				"nickname": "wangwu",
				"sex": "女",
				"age": 40
			},
			"sort": [40, 1.0]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1001",
			"_score": 1.0,
			"_source": {
				"name": "zhangsan",
				"nickname": "zhangsan",
				"sex": "男",
				"age": 30
			},
			"sort": [30, 1.0]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1005",
			"_score": 1.0,
			"_source": {
				"name": "zhangsan2",
				"nickname": "zhangsan2",
				"sex": "女",
				"age": 30
			},
			"sort": [30, 1.0]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1002",
			"_score": 1.0,
			"_source": {
				"name": "lisi",
				"nickname": "lisi",
				"sex": "男",
				"age": 20
			},
			"sort": [20, 1.0]
		}]
	}
}
```

##### **13) 高亮查询**

在进行关键字搜索时，搜索出的内容中的关键字会显示不同的颜色，称之为高亮。  

Elasticsearch 可以对查询内容中的关键字部分，进行标签和样式(高亮)的设置。在使用 match 查询的同时，加上一个 highlight 属性：

* pre_tags：前置标签
* post_tags：后置标签
* fields：需要高亮的字段
* title：这里声明 title 字段需要高亮，后面可以为这个字段设置特有配置， 也可以空

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "match": {
            "name": "zhangsan"
        }
    },
    "highlight": {
        "pre_tags": "<font color='red'>",
        "post_tags": "</font>",
        "fields": {
            "name": {}
        }
    }
}
```

服务器响应结果：

```json
{
    "took": 72,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.3862942,
        "hits": [
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "1001",
                "_score": 1.3862942,
                "_source": {
                    "name": "zhangsan",
                    "nickname": "zhangsan",
                    "sex": "男",
                    "age": 30
                },
                "highlight": {  // 高亮显示
                    "name": [
                        "<font color='red'>zhangsan</font>"
                    ]
                }
            }
        ]
    }
}
```

##### **14) 分页查询**

from：当前页的起始索引，默认从 0 开始。 `from = (pageNum - 1) * size`

size：每页显示多少条

在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "query": {
        "match_all": {}
    },
    "sort": [
        {
            "age": {
                "order": "desc"
            }
        }
    ],
    "from": 0,
    "size": 2
}
```

服务器响应结果：  

```json
{
	"took": 2,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 5,
			"relation": "eq"
		},
		"max_score": null,
		"hits": [{
			"_index": "student",
			"_type": "_doc",
			"_id": "1004",
			"_score": null,
			"_source": {
				"name": "zhangsan1",
				"nickname": "zhangsan1",
				"sex": "女",
				"age": 50
			},
			"sort": [50]
		}, {
			"_index": "student",
			"_type": "_doc",
			"_id": "1003",
			"_score": null,
			"_source": {
				"name": "wangwu",
				"nickname": "wangwu",
				"sex": "女",
				"age": 40
			},
			"sort": [40]
		}]
	}
}
```

#### 2.2.5 HTTP 操作之聚合查询

聚合允许使用者对 es 文档进行统计分析，类似与关系型数据库中的 group by，当然还有很多其他的聚合，例如取最大值、平均值等等。

##### **1) 最大值查询**

对某个字段取最大值 max，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search  

```json
{
    "aggs":{
        "max_age":{
            "max":{"field":"age"}
        }
    },
    "size":0
}
```

服务器响应结果： 

```json
{
    "took": 12,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "max_age": {
            "value": 50.0
        }
    }
}
```

##### **2) 最小值查询**

对某个字段取最小值 min，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "aggs":{
        "min_age":{
            "min":{"field":"age"}
        }
    },
    "size":0
}
```

服务器响应结果：  

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "min_age": {
            "value": 20.0
        }
    }
}
```

##### **3) 求和**

对某个字段求和 sum，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
	"aggs": {
		"sum_age": {
			"sum": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```

服务器响应结果：  

```json
{
    "took": 5,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "sum_age": {
            "value": 170.0
        }
    }
}
```

##### **4) 求平均值**

对某个字段取平均值 avg，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "aggs":{
        "avg_age":{
            "avg":{"field":"age"}
        }
    },
    "size":0
}
```

服务器响应结果：  

```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "avg_age": {
            "value": 34.0
        }
    }
}
```

##### **5) 去重**

对某个字段的值进行去重之后再取总数，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search  

```json
{
    "aggs":{
        "distinct_age":{
            "cardinality":{"field":"age"}
        }
    },
    "size":0
}
```

服务器响应结果：

```json
{
    "took": 10,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "distinct_age": {
            "value": 4
        }
    }
}
```

##### **6) State 聚合**

State 聚合：对某个字段一次性返回 count， max， min， avg 和 sum 五个指标，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "aggs":{
        "stats_age":{
            "stats":{"field":"age"}
        }
    },
    "size":0
}
```

服务器响应结果：

```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "stats_age": {
            "count": 5,
            "min": 20.0,
            "max": 50.0,
            "avg": 34.0,
            "sum": 170.0
        }
    }
}
```

##### **7) 桶聚合查询**

桶聚和相当于 sql 中的 group by 语句

* terms 聚合，分组统计，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "aggs":{
        "age_groupby":{
            "terms":{"field":"age"}
        }
    },
    "size":0
}
```

服务器响应结果：  

```json
{
    "took": 4,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "age_groupby": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 30,
                    "doc_count": 2
                },
                {
                    "key": 20,
                    "doc_count": 1
                },
                {
                    "key": 40,
                    "doc_count": 1
                },
                {
                    "key": 50,
                    "doc_count": 1
                }
            ]
        }
    }
}
```

* 在 terms 分组下再进行聚合，在 Postman 中，向 ES 服务器发 GET 请求 ： http://127.0.0.1:9200/student/_search

```json
{
    "aggs":{
        "age_groupby":{
            "terms":{"field":"age"}
        }
    },
    "size":0
}
```

服务器响应结果：  

```json
{
    "took": 8,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "age_groupby": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 30,
                    "doc_count": 2
                },
                {
                    "key": 20,
                    "doc_count": 1
                },
                {
                    "key": 40,
                    "doc_count": 1
                },
                {
                    "key": 50,
                    "doc_count": 1
                }
            ]
        }
    }
}
```

### 2.3 Java API 操作

Elasticsearch 软件是由 Java 语言开发的，所以也可以通过 Java API 的方式对 Elasticsearch 服务进行访问

**创建 Maven 项目，导入依赖**

```xml
<groupId>cn.xyc</groupId>
<artifactId>es-demo</artifactId>
<version>1.0-SNAPSHOT</version>

<properties>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
</properties>
<dependencies>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch的客户端 -->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch依赖2.x的log4j -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.9</version>
    </dependency>
    <!-- junit单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

#### 2.3.1 客户端对象

这里采用高级 REST 客户端对象：

```java
package cn.xyc.es;

import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;

public class ES01Client {

    public static void main(String[] args) throws Exception {

        // 创建客户端对象
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http"))
        );

        // 关闭客户端连接
        client.close();
    }
}
```

> 注意： 9200 端口为 Elasticsearch 的 Web 通信端口， localhost 为启动 ES 服务的主机名

#### 2.2.2 索引操作

ES 服务器正常启动后，可以通过 Java API 客户端对象对 ES 索引进行操作  

##### 1) 创建索引

```java
// 创建客户端对象
RestHighLevelClient client = new RestHighLevelClient(
    RestClient.builder(new HttpHost("localhost", 9200, "http"))
);

CreateIndexRequest request = new CreateIndexRequest("user");
// 发送请求，获取响应
CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
boolean acknowledged = response.isAcknowledged();

// 响应状态
System.out.println("操作状态 = " + acknowledged);

// 关闭客户端连接
client.close();
```

##### 2) 查看索引

```java
// 创建客户端对象
RestHighLevelClient client = new RestHighLevelClient(
    RestClient.builder(new HttpHost("localhost", 9200, "http"))
);

// 查看索引
GetIndexRequest request = new GetIndexRequest("user");
// 发送请求，获取响应
GetIndexResponse response = client.indices().get(request, RequestOptions.DEFAULT);
// 查看结果
System.out.println("aliases:"+response.getAliases());
System.out.println("mappings:"+response.getMappings());
System.out.println("settings:"+response.getSettings());

// 关闭客户端连接
client.close();
```

##### 3) 删除索引

```java
// 创建客户端对象
RestHighLevelClient client = new RestHighLevelClient(
    RestClient.builder(new HttpHost("localhost", 9200, "http"))
);

// 删除索引 - 请求对象
DeleteIndexRequest request = new DeleteIndexRequest("student_new");
// 发送请求，获取响应
AcknowledgedResponse response = client.indices().delete(request, RequestOptions.DEFAULT);
// 查看结果
System.out.println("操作结果 ： " + response.isAcknowledged());

// 关闭客户端连接
client.close();
```

#### 2.2.3 文档操作

创建数据模型

```java
package cn.xyc.es.domain;

import java.io.Serializable;

public class User implements Serializable {

    private String name;
    private Integer age;
    private String sex;

    // get/set
}
```

##### 1) 新增文档

创建数据，添加到文档中

```java
// 创建客户端对象
RestHighLevelClient client = new RestHighLevelClient(
    RestClient.builder(new HttpHost("localhost", 9200, "http"))
);

// 新增文档 - 请求对象
IndexRequest request = new IndexRequest();
// 设置索引及唯一性标识
request.index("user").id("1001");
// 创建数据对象
User user = new User();
user.setName("zhangsan");
user.setAge(30);
user.setSex("男");

ObjectMapper objectMapper = new ObjectMapper();
String jsonUser = objectMapper.writeValueAsString(user);
// 添加文档数据，数据格式为 JSON 格式
request.source(jsonUser, XContentType.JSON);
// 客户端发送请求，获取响应对象
IndexResponse response = client.index(request, RequestOptions.DEFAULT);
// 打印结果信息
System.out.println("_index:" + response.getIndex());
System.out.println("_id:" + response.getId());
System.out.println("_result:" + response.getResult());

// 关闭客户端连接
client.close();
```

##### 2) 修改文档

```java
// 创建客户端对象
RestHighLevelClient client = new RestHighLevelClient(
    RestClient.builder(new HttpHost("localhost", 9200, "http"))
);

// 修改文档 - 请求对象
UpdateRequest request = new UpdateRequest();
// 配置修改参数
request.index("user").id("1001");
// 设置请求体，对数据进行修改
request.doc(XContentType.JSON, "sex", "女");
// 客户端发送请求，获取响应对象
UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
// 打印结果信息
System.out.println("_index:" + response.getIndex());
System.out.println("_id:" + response.getId());
System.out.println("_result:" + response.getResult());

// 关闭客户端连接
client.close();
```

##### 3) 查询文档

```java
// 查询文档
// 创建请求对象
GetRequest request = new GetRequest().index("user").id("1001");
// 客户端发送请求，获取响应对象
GetResponse response = client.get(request, RequestOptions.DEFAULT);
// 打印结果信息
System.out.println("_index:" + response.getIndex());
System.out.println("_type:" + response.getType());
System.out.println("_id:" + response.getId());
System.out.println("source:" + response.getSourceAsString());
```

##### 4) 删除文档

```java
// 删除文档
// 创建请求对象
DeleteRequest request = new DeleteRequest().index("user").id("1");
// 客户端发送请求，获取响应对象
DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
// 打印信息
System.out.println(response.toString());
```

##### 5) 批量操作

* 批量新增：

```java
// 批量操作-批量新增：
BulkRequest request = new BulkRequest();
request.add(new IndexRequest().index("user").id("1001").source(XContentType.JSON, "name", "zhangsan", "age", "30", "sex", "男"));
request.add(new IndexRequest().index("user").id("1002").source(XContentType.JSON, "name", "lisi", "age", "30", "sex", "男"));
request.add(new IndexRequest().index("user").id("1003").source(XContentType.JSON, "name", "wangwu", "age", "40", "sex", "女"));
request.add(new IndexRequest().index("user").id("1004").source(XContentType.JSON, "name", "zhaoliu", "age", "50", "sex", "男"));
request.add(new IndexRequest().index("user").id("1005").source(XContentType.JSON, "name", "liuneng", "age", "30", "sex", "女"));
// 客户端发送请求，获取响应对象
BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
// 打印结果信息
System.out.println("took:" + responses.getTook());
System.out.println("items:" + responses.getItems());
```

* 批量删除：

```java
// 批量删除
// 创建批量删除请求对象
BulkRequest request = new BulkRequest();
request.add(new DeleteRequest().index("user").id("1001"));
request.add(new DeleteRequest().index("user").id("1002"));
request.add(new DeleteRequest().index("user").id("1003"));
request.add(new DeleteRequest().index("user").id("1004"));
request.add(new DeleteRequest().index("user").id("1005"));
// 客户端发送请求，获取响应对象
BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
// 打印结果信息
System.out.println("took:" + responses.getTook());
System.out.println("items:" + responses.getItems());
```

#### 2.2.4 高级查询

##### 1) 请求体查询：查询所有索引数据

```java
// 创建客户端对象
RestHighLevelClient client = new RestHighLevelClient(
    RestClient.builder(new HttpHost("localhost", 9200, "http"))
);

// 查询所有索引数据
SearchRequest request = new SearchRequest();
request.indices("user");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
// 查询所有数据
sourceBuilder.query(QueryBuilders.matchAllQuery());
request.source(sourceBuilder);

// 客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    //输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");

// 关闭客户端连接
client.close();
```

##### 2) 请求体查询：term 查询

```java
// term 查询，查询条件为关键字
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("user");
// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.termQuery("age", "30"));
request.source(sourceBuilder);

// 客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    //输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 3) 请求体查询：分页查询

```java
// 分页查询
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("user");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());
// 分页查询
// 当前页其实索引(第一条数据的顺序号)， from
sourceBuilder.from(0);
// 每页显示多少条 size
sourceBuilder.size(2);
request.source(sourceBuilder);

// 客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    //输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 4) 请求体查询：数据排序

```java
// 数据排序
// 查询所有索引数据
SearchRequest request = new SearchRequest();
request.indices("user");
// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());
// 排序
sourceBuilder.sort("age", SortOrder.ASC);

request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    //输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 5) 请求体查询：过滤字段  

```java
// 过滤字段
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("user");
// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());

//查询字段过滤
String[] excludes = {};
String[] includes = {"name", "age"};
sourceBuilder.fetchSource(includes, excludes);

request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    //输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 6) 请求体查询：bool查询  

```java
// Bool 查询
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("user");
// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();

// 必须包含
boolQueryBuilder.must(QueryBuilders.matchQuery("age", "30"));
// 一定不含
boolQueryBuilder.mustNot(QueryBuilders.matchQuery("name", "zhangsan"));
// 可能包含
boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "男"));

sourceBuilder.query(boolQueryBuilder);
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    //输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 7) 请求体查询：范围查询  

```java
// 范围查询
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("user");
// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("age");
// 大于等于
rangeQuery.gte("30");
// 小于等于
rangeQuery.lte("40");

sourceBuilder.query(rangeQuery);
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    //输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 8) 请求体查询：模糊查询

```java
// 模糊查询
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("user");
// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.fuzzyQuery("name","zhangsan").fuzziness(Fuzziness.ONE));
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    // 输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 9) 请求体查询：高亮查询

```java
// 高亮查询
// 高亮查询
SearchRequest request = new SearchRequest().indices("student");
// 创建查询请求体构建器
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
// 构建查询方式：高亮查询
TermsQueryBuilder termsQueryBuilder = QueryBuilders.termsQuery("name","zhangsan");
// 设置查询方式
sourceBuilder.query(termsQueryBuilder);
// 构建高亮字段
HighlightBuilder highlightBuilder = new HighlightBuilder();
highlightBuilder.preTags("<font color='red'>");//设置标签前缀
highlightBuilder.postTags("</font>");//设置标签后缀
highlightBuilder.field("name");//设置高亮字段
// 设置高亮构建对象
sourceBuilder.highlighter(highlightBuilder);
// 设置请求体
request.source(sourceBuilder);
// 客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
    String sourceAsString = hit.getSourceAsString();
    System.out.println(sourceAsString);
    // 打印高亮结果
    Map<String, HighlightField> highlightFields = hit.getHighlightFields();
    System.out.println(highlightFields);
}
System.out.println("<<========");
```

##### 10) 请求体查询：聚合查询

```java
// 聚合查询之：最大年龄
SearchRequest request = new SearchRequest().indices("user");
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.aggregation(AggregationBuilders.max("maxAge").field("age"));
// 设置请求体
request.source(sourceBuilder);
// 客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 打印响应结果
System.out.println(response);

// 聚合查询之：最大年龄
SearchRequest request = new SearchRequest().indices("user");
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.aggregation(AggregationBuilders.terms("age_groupby").field("age"));
// 设置请求体
request.source(sourceBuilder);
//客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
//打印响应结果
SearchHits hits = response.getHits();
System.out.println(response);
```

## 3. Elasticsearch 环境

### 3.1 相关概念

#### 3.1.1 单机 & 集群

单台 Elasticsearch 服务器提供服务，往往都有最大的负载能力，超过这个阈值，服务器性能就会大大降低甚至不可用，所以生产环境中，一般都是运行在指定服务器集群中。

除了负载能力，单点服务器也存在其他问题：

* 单台机器存储容量有限
* 单服务器容易出现单点故障，无法实现高可用
* 单服务的并发处理能力有限

配置服务器集群时，集群中节点数量没有限制，大于等于 2 个节点就可以看做是集群了。一般出于高性能及高可用方面来考虑集群中节点数量都是 3 个以上。

#### 3.1.2 集群 Cluster

一个集群就是由一个或多个服务器节点组织在一起，共同持有整个的数据，并一起提供索引和搜索功能。一个 Elasticsearch 集群有一个唯一的名字标识，这个名字默认就是 ”elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

#### 3.1.3 节点 Node

集群中包含很多服务器， 一个节点就是其中的一个服务器。 作为集群的一部分，它存储数据，参与集群的索引和搜索功能。

一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于 Elasticsearch 集群中的哪些节点。

一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫做“elasticsearch”的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中。  

在一个集群里，只要你想，可以拥有任意多个节点。而且，如果当前你的网络中没有运行任何 Elasticsearch 节点，这时启动一个节点，会默认创建并加入一个叫做“elasticsearch”的集群。  

### 3.2 Windows 集群

第一步：将 es 的压缩包解压三份至随意创建的目录中，如创建文件夹`elasticsearch-cluster`，然后解压缩 es 软件，分别命名为：`elasticsearch-1`，`elasticsearch-2`，`elasticsearch-1`；

第二步：修改集群文件目录中每个节点的 `config/elasticsearch.yml` 配置文件  

1. node-1节点：

   ```yml
   # 集群名称，节点之间要保持一致
   cluster.name: my-application
   
   # 节点名称，集群内要唯一
   node.name: node-1
   node.master: true
   node.data: true
   
   #ip 地址
   network.host: localhost
   #http 端口
   http.port: 9201
   #tcp 监听端口
   transport.tcp.port: 9301
   
   # 服务被发现
   discovery.seed_hosts: ["127.0.0.1:9301", "127.0.0.1:9302","127.0.0.1:9303"]
   discovery.zen.fd.ping_timeout: 1m
   discovery.zen.fd.ping_retries: 5
   
   # 集群内的可以被选为主节点的节点列表
   cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
   
   # 跨域配置
   # action.destructive_requires_name: true
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   ```

2. node-2节点：

   ```yaml
   # 集群名称，节点之间要保持一致
   cluster.name: my-application
   
   # 节点名称，集群内要唯一
   node.name: node-2
   node.master: true
   node.data: true
   
   #ip 地址
   network.host: localhost
   #http 端口
   http.port: 9202
   #tcp 监听端口
   transport.tcp.port: 9302
   
   # 后续内容一致
   ```

3. node-3节点：

   ```yaml
   # 集群名称，节点之间要保持一致
   cluster.name: my-application
   
   # 节点名称，集群内要唯一
   node.name: node-3
   node.master: true
   node.data: true
   
   #ip 地址
   network.host: localhost
   #http 端口
   http.port: 9203
   #tcp 监听端口
   transport.tcp.port: 9303
   
   # 后续内容一致
   ```

第三步：启动集群

1. 启动前先删除每个节点中的 data 目录中所有内容（如果存在）  
2. 分别双击执行 bin/elasticsearch.bat, 启动节点服务器，启动后，会自动加入指定名称的集群；
3. 等待启动完成

第四步：测试集群-测试状态

1. 查看node-1状态：

   ```json
   GET  http://127.0.0.1:9201/_cluster/health
   
   {
       "cluster_name": "my-application",
       "status": "green",
       "timed_out": false,
       "number_of_nodes": 3,  // 有3个节点
       "number_of_data_nodes": 3,  // 有3个数据节点
       "active_primary_shards": 0,
       "active_shards": 0,
       "relocating_shards": 0,
       "initializing_shards": 0,
       "unassigned_shards": 0,
       "delayed_unassigned_shards": 0,
       "number_of_pending_tasks": 0,
       "number_of_in_flight_fetch": 0,
       "task_max_waiting_in_queue_millis": 0,
       "active_shards_percent_as_number": 100.0
   }
   ```

2. 查看node-1状态：

   ```json
   GET  http://127.0.0.1:9202/_cluster/health
   
   // 输出相同
   ```

3. 查看node-3状态：

   ```json
   GET  http://127.0.0.1:9203/_cluster/health
   
   // 输出相同
   ```

第五步：测试集群-添加索引

1. 向集群中的 node-1 节点增加索引

   ```json
   PUT http://127.0.0.1:9201/user
   
   // 响应
   {
       "acknowledged": true,
       "shards_acknowledged": true,
       "index": "user"
   }
   ```

2. 向集群中的 node-2 节点查询索引  

   ```json
   GET http://127.0.0.1:9202/user
   
   // 响应
   {
       "user": {
           "aliases": {},
           "mappings": {},
           "settings": {
               "index": {
                   "creation_date": "1638254793261",
                   "number_of_shards": "1",
                   "number_of_replicas": "1",
                   "uuid": "LcLZ3DIqT3WSOUdcSxfffA",
                   "version": {
                       "created": "7080099"
                   },
                   "provided_name": "user"
               }
           }
       }
   }
   ```

可见，集群搭建ok