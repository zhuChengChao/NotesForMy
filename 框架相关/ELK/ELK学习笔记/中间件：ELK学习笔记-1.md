# 					中间件：ELK学习笔记-1

> 很早之前粗略的学习过Lucene和Elasticsearch，然后最近实习过程中需要用到一些ES的基础操作，虽说使用起来没啥问题，但还是想稍微系统的学习一下，也能在秋招的时候稍微说道说道。
>
> 在 bilibili 上搜了一个课程，课程地址：[黑马：ELK高级搜索](https://www.bilibili.com/video/BV1Nt4y1m7qL)，跟着视频也做了些笔记，总的分了7篇，此为1/7篇。
>
> > 学的很草率...空了再来完善:cry:

## 1. 概述

ELK 是包含但不限于 Elasticsearch（ES）、Logstash、Kibana 三个开源软件的组成的一个整体。这三个软件合成 ELK。是用于数据抽取 Logstash、搜索分析 Elasticsearch、数据展现 Kibana 的一整套解决方案，所以也称作 ELK Stack。

该教程分别对三个组件进行详细介绍，尤其是 Elasticsearch，因为它是 ELK 的核心。从 ES 底层对文档、索引、搜索、聚合、集群进行介绍，从搜索和聚合分析实例来展现 ES 的魅力。Logstash 从内部如何采集数据到指定地方来展现它数据采集的功能。Kibana 则从数据绘图展现数据可视化的功能。

## 2. Elastic Stack简介

### 2.1 简介

ELK 是一个免费开源的日志分析架构技术栈总称，官网https://www.elastic.co/cn。包含三大基础组件，分别是Elasticsearch、Logstash、Kibana。但实际上ELK不仅仅适用于日志分析，它还可以支持其它任何数据搜索、分析和收集的场景，日志分析和收集只是更具有代表性，并非唯一性。

随着 ELK 的发展，又有新成员Beats、Elastic Cloud 的加入，所以就形成了 Elastic Stack。所以说，ELK 是旧的称呼，Elastic Stack是新的名字。

###  2.2 特色

- 处理方式灵活：ES 是目前最流行的**准实时全文检索引擎**，具有高速检索大数据的能力。

- 配置简单：安装 ELK 的每个组件，仅需配置每个组件的一个配置文件即可。修改处不多，因为大量参数已经默认配在系统中，修改想要修改的选项即可。

- 接口简单：采用 json 形式 Restful API 接受数据并响应，无关语言。

- 性能高效：ES 基于优秀的全文搜索技术 Lucene，采用**倒排索引**，可以轻易地在百亿级别数据量下，搜索出想要的内容，并且是秒级响应。

- 灵活扩展：ES 和 Logstash 都可以根据集群规模线性拓展，ES 内部自动实现集群协作。

- 数据展现华丽：Kibana 作为前端展现工具，图表华丽，配置简单。

###  2.3 组件介绍

**Elasticsearch**

ES 是使用 Java 开发，基于Lucene、分布式、通过 Restful 方式进行交互的近实时搜索平台框架。它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，Restful 风格接口，多数据源，自动搜索负载等。

**Logstash**

Logstash 基于 Java 开发，是一个数据抽取转化工具。一般工作方式为 C/S 架构，Client 端安装在需要收集信息的主机上，Server 端负责将收到的各节点日志进行过滤、修改等操作在一并发往 Elasticsearch 或其他组件上去。 

**Kibana**

Kibana 基于 node.js，也是一个开源和免费的可视化工具。Kibana 可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以汇总、分析和搜索重要数据日志。

**Beats**

Beats 平台集合了多种单一用途数据采集器。它们从成百上千或成千上万台机器和系统向 Logstash 或 Elasticsearch 发送数据。

Beats 由如下组成:

* Packetbeat：轻量型网络数据采集器，用于深挖网线上传输的数据，了解应用程序动态。Packetbeat 是一款轻量型网络数据包分析器，能够将数据发送至 Logstash 或 Elasticsearch。其支持 ICMP (v4 and v6)、DNS、HTTP、MySQL、PostgreSQL、Redis、MongoDB、Memcache等协议。

* Filebeat：轻量型日志采集器。当您要面对成百上千、甚至成千上万的服务器、虚拟机和容器生成的日志时，请告别 SSH 吧。Filebeat 将为您提供一种轻量型方法，用于转发和汇总日志与文件，让简单的事情不再繁杂。

* Metricbeat ：轻量型指标采集器。Metricbeat 能够以一种轻量型的方式，输送各种系统和服务统计数据，从 CPU 到内存，从 Redis 到 Nginx，不一而足。可定期获取外部系统的监控指标信息，其可以监控、收集 Apache HTTP、HAProxy、MongoDB、MySQL、Nginx、PostgreSQL、Redis、System、Zookeeper等服务。

* Winlogbeat：轻量型 Windows 事件日志采集器。用于密切监控基于 Windows 的基础设施上发生的事件。Winlogbeat 能够以一种轻量型的方式，将 Windows 事件日志实时地流式传输至 Elasticsearch 和 Logstash。

* Auditbeat：轻量型审计日志采集器。收集您 Linux 审计框架的数据，监控文件完整性。Auditbeat 实时采集这些事件，然后发送到 Elastic Stack 其他部分做进一步分析。

* Heartbeat：面向运行状态监测的轻量型采集器。通过主动探测来监测服务的可用性。通过给定 URL 列表，Heartbeat 仅仅询问：网站运行正常吗？Heartbeat 会将此信息和响应时间发送至 Elastic 的其他部分，以进行进一步分析。

* Functionbeat：面向云端数据的无服务器采集器。在作为一项功能部署在云服务提供商的功能即服务 (FaaS) 平台上后，Functionbeat 即能收集、传送并监测来自您的云服务的相关数据。

**Elastic Cloud**

基于 Elasticsearch 的软件即服务 SaaS 解决方案。通过 Elastic 的官方合作伙伴使用托管的 Elasticsearch 服务。

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415021.png" alt="ElasticCloud" style="zoom:67%;" />

## 3. ElasticSearch是什么

### 3.1 搜索是什么

概念：用户输入想要的关键词，返回含有该关键词的所有信息。

场景：

1. 互联网搜索：谷歌、百度、各种新闻首页
2. 站内搜索（垂直搜索）：企业 OA 查询订单、人员、部门；电商网站内部搜索商品（淘宝、京东）场景。

### 3.2 数据库搜索弊端

站内搜索（垂直搜索）：数据量小，简单搜索，可以使用数据库。

问题出现：

1. 存储问题：电商网站商品上亿条时，涉及到单表数据过大必须拆分表，数据库磁盘占用过大必须分库（MyCat）。

2. 性能问题：解决上面问题后，查询“笔记本电脑”等关键词时，上亿条数据的商品名字段逐行扫描，性能跟不上。

3. 不能分词：如搜索“笔记本电脑”，只能搜索完全和关键词一样的数据，那么数据量小时，搜索“笔记电脑”，“电脑”数据要不要给用户。

**互联网搜索，肯定不会使用数据库搜索，数据量太大，PB级。**

### 3.3 全文检索&倒排索引&Lucene

**全文检索**，倒排索引。数据存储时，进行分词建立 term 索引库。

倒排索引源于实际应用中需要**根据属性的值来查找记录**。这种索引表中的每一项都包括一个属性值和具有该属性值的各记录的地址。由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因而称为倒排索引(inverted index)。带有倒排索引的文件我们称为倒排索引文件，简称倒排文件(inverted file)。

**Lucene**：就是一个 jar 包，里面封装了全文检索的引擎、搜索的算法代码。开发时，引入 Lucene 的jar包，通过 API 开发搜索相关业务，底层会在磁盘建立索引库。

### 3.4 什么是ES

简介

![ES简介1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261414484.png)

官网：https://www.elastic.co/cn/products/elasticsearch

![ES简介2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261414814.png)

**ES的功能**

- 分布式的搜索引擎和数据分析引擎

  搜索：互联网搜索、电商网站站内搜索、OA系统查询

  数据分析：电商网站查询近一周哪些品类的图书销售前十；新闻网站，最近3天阅读量最高的十个关键词，舆情分析。

- 全文检索，结构化检索，数据分析

  全文检索：搜索商品名称包含 Java 的图书 `select * from books where book_name like "%java%"`

  结构化检索：搜索商品分类为 spring 的图书都有哪些，`select * from books where category_id='spring'`

  数据分析：分析每一个分类下有多少种图书，`select category_id, count(*) from books group by category_id`

- 对海量数据进行近实时的处理

  分布式：ES自动可以将海量数据分散到多台服务器上去存储和检索，进行并行查询，提高搜索效率。相对的，Lucene是单机应用。

  近实时：数据库上亿条数据查询，搜索一次耗时几个小时，是批处理（batch-processing）。而es只需秒级即可查询海量数据，所以叫近实时，秒级。

**ES的使用场景**

国外：

- 维基百科，类似百度百科，“网络七层协议”的维基百科，全文检索，高亮，搜索推荐。

- Stack Overflow（国外的程序讨论论坛），相当于程序员的贴吧。遇到it问题去上面发帖，热心网友下面回帖解答。

- GitHub（开源代码管理），搜索上千亿行代码。

- 电商网站，检索商品

- 日志数据分析，Logstash采集日志，ES进行复杂的数据分析（ELK技术，ES+Logstash+Kibana）

- 商品价格监控网站，用户设定某商品的价格阈值，当低于该阈值的时候，发送通知消息给用户，比如说订阅《java编程思想》的监控，如果价格低于27块钱，就通知我，我就去买。

- BI系统，商业智能（Business Intelligence）。大型连锁超市，分析全国网点传回的数据，分析各个商品在什么季节的销售量最好、利润最高。成本管理，店面租金、员工工资、负债等信息进行分析。从而部署下一个阶段的战略目标。

国内：

- 百度搜索，第一次查询，使用ES。

- OA、ERP系统站内搜索。

**ES的特点**

- 可拓展性：大型分布式集群（数百台服务器）技术，处理PB级数据，大公司可以使用。小公司数据量小，也可以部署在单机。大数据领域使用广泛。

- 技术整合：将全文检索、数据分析、分布式相关技术整合在一起：Lucene（全文检索），商用的数据分析软件（BI软件），分布式数据库（MyCat）

- 部署简单：开箱即用，很多默认配置不需关心，解压完成直接运行即可。拓展时，只需多部署几个实例即可，负载均衡、分片迁移集群内部自己实施。

- 接口简单：使用 Restful API 经行交互，跨语言。

- 功能强大：Elasticsearch 作为传统数据库的一个补充，提供了数据库所不不能提供的很多功能，如全文检索，同义词处理，相关度排名。

![可扩展性](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415051.png)

![相关度](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415378.png)

![弹性](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415742.png)

### 3.5 ES核心概念

#### 3.5.1 Lucene和ES的关系

Lucene：最先进、功能最强大的搜索库，直接基于 Lucene 开发，非常复杂，API 复杂

Elasticsearch：基于Lucene，封装了许多 Lucene 底层功能，提供简单易用的 Restful API接口和许多语言的客户端，如 Java 的高级客户端（Java High Level REST Client）和底层客户端（Java Low Level REST Client）

![开发客户端](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415895.png)

> 起源：Shay Banon。2004年失业，陪老婆去伦敦学习厨师。失业在家帮老婆写一个菜谱搜索引擎。封装了 Lucene 的开源项目，compass。找到工作后，做分布式高性能项目，再封装 compass，写出了 Elasticsearch，使得 Lucene 支持分布式。

#### 3.5.2 ES 的核心概念

1. **NRT（Near Realtime）：近实时**，体现在两方面：

   1. 写入数据时，过1秒才会被搜索到，因为内部在分词、录入索引。
   2. ES 搜索时：搜索和分析数据需要秒级出结果。

2. **Cluster：集群**

   包含一个或多个启动着 ES 实例的机器群。通常一台机器起一个 ES 实例。同一网络下，集名一样的多个 ES 实例自动组成集群，自动均衡分片等行为。默认集群名为 “elasticsearch”。

3. **Node：节点**

   每个 ES 实例称为一个节点。节点名自动分配，也可以手动配置。

4. **Index：索引**

   包含一堆有相似结构的文档数据。

   索引创建规则：

   - 仅限小写字母

   - 不能包含 `\`、`/`、 `*`、`?`、`"`、`<`、`>`、`|`、`#` 以及空格符等特殊符号

   - 从 7.0 版本开始不再包含冒号

   - 不能以 `-`、`_`或`+`开头

   - 不能超过255个字节（注意它是字节，因此多字节字符将计入255个限制）

5. **Document：文档**

   ES 中的最小数据单元。**一个 document 就像数据库中的一条记录**。通常以 json 格式显示。多个 document 存储于一个索引（Index）中。

   ```json
   book document
   {
     "book_id": "1",
     "book_name": "java编程思想",
     "book_desc": "从Java的基础语法到最高级特性（深入的[面向对象](https://baike.baidu.com/item/面向对象)概念、多线程、自动项目构建、单元测试和调试等），本书都能逐步指导你轻松掌握。",
     "category_id": "2",
     "category_name": "java"
   }
   ```
   
6. **Field：字段**

   就像数据库中的列（columns），定义每个 document 应该有的字段。

7. **Type：类型**

   每个索引里都可以有一个或多个 type，type 是 index 中的一个逻辑数据分类，一个 type 下的 document，都有相同的 field。

   **注意**：6.0 之前的版本有 type（类型）概念，type 相当于关系数据库的表，ES 官方将在 ES9.0 版本中彻底删除 type。**本教程 type 都为 _doc**。

8. **shard：分片**

   index 数据过大时，将 index 里面的数据，分为多个 shard，分布式的存储在各个服务器上面。可以支持海量数据和高并发，提升性能和吞吐量，充分利用多台机器的 CPU。

9. **replica：副本**

   在分布式环境下，任何一台机器都会随时宕机，如果宕机，index 的一个分片没有，导致此 index 不能搜索。所以，为了保证数据的安全，我们会将每个 index 的分片经行备份，存储在另外的机器上。保证少数机器宕机es集群仍可以搜索。

   能正常提供查询和插入的分片我们叫做主分片（primary shard），其余的我们就管他们叫做备份的分片（replica shard）。

   ES6 默认新建索引时，5分片，2副本，也就是一主一备，共10个分片。所以，es集群最小规模为两台。

#### 3.5.3 ES vs 数据库概念

| **关系型数据库（比如MySQL）** | **非关系型数据库（ES）** |
| ----------------------------- | ------------------------ |
| 数据库Database                | 索引Index                |
| 表Table                       | 索引Index（原为Type）    |
| 数据行Row                     | 文档Document             |
| 数据列Column                  | 字段Field                |
| 约束Schema                    | 映射Mapping              |

## 4. ES相关软件安装

> 这里所有关于ElasticStack版本我用的都是7.8.0

### 4.1 Windows安装ES

> 下载地址：https://www.elastic.co/cn/downloads/elasticsearch

下载和解压缩Elasticsearch安装包，查看目录结构。                                                  

* `bin`：脚本目录，包括：启动、停止等可执行脚本
* `config`：配置文件目录
* `data`：索引目录，存放索引文件的地方
* `logs`：日志目录
* `modules`：模块目录，包括了ES功能模块
* `plugins` :插件目录，ES支持插件机制

**配置文件**：ES的配置文件的地址根据安装形式的不同而不同，

* 使用zip、tar安装，配置文件的地址在安装目录的config下。

* 使用RPM安装，配置文件在`/etc/elasticsearch`下。

* 使用MSI安装，配置文件的地址在安装目录的config下，并且会自动将config目录地址写入环境变量ES_PATH_CONF。

#### 4.1.1 配置文件1：elasticsearch.yml

配置格式是 YAML，可以采用如下两种方式：

* 方式1：层次方式

```yaml
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```

* 方式2：属性方式

```yaml
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

**常用的配置项如下**

```yaml
# 配置elasticsearch的集群名称，默认是elasticsearch。建议修改成一个有意义的名称。
cluster.name: 	
# 节点名，通常一台物理服务器就是一个节点，es会默认随机指定一个名字，建议指定一个有意义的名称，方便管理
# 一个或多个节点组成一个cluster集群，集群是一个逻辑的概念，节点是物理概念，后边章节会详细介绍。
node.name:
# 设置配置文件的存储路径，tar或zip包安装默认在es根目录下的config文件夹，rpm安装默认在/etc/ elasticsearch
path.conf: 
# 设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开。
path.data:
# 设置日志文件的存储路径，默认是es根目录下的logs文件夹
path.logs:
# 设置插件的存放路径，默认是es根目录下的plugins文件夹
path.plugins: 
# 设置为true可以锁住ES使用的内存，避免内存与swap分区交换数据。	
bootstrap.memory_lock: true
# 设置绑定主机的ip地址，设置为0.0.0.0表示绑定任何ip，允许外网访问，生产环境建议设置为具体的ip。
network.host: 
# 设置对外服务的http端口，默认为9200。
http.port: 9200
# 集群结点之间通信端口
transport.tcp.port: 9300  
# 指定该节点是否有资格被选举成为master结点，默认是true，如果原来的master宕机会重新选举新的master。
node.master: 
# 指定该节点是否存储索引数据，默认为true。
node.data: 
# 设置集群中master节点的初始列表。	
discovery.zen.ping.unicast.hosts: ["host1:port", "host2:port", "..."]
# 设置ES自动发现节点连接超时的时间，默认为3秒，如果网络延迟高可设置大些。
discovery.zen.ping.timeout: 3s
# 主结点数量的最少值 ,此值的公式为：(master_eligible_nodes / 2) + 1 ，比如：有3个符合要求的主结点，那么这里要设置为2。
discovery.zen.minimum_master_nodes:
# 单机允许的最大存储结点数，通常单机启动一个结点建议设置为1，开发环境如果单机启动多个节点可设置大于1。
node.max_local_storage_nodes: 
```

#### 4.1.2 配置文件2：jvm.options

设置最小及最大的JVM堆内存大小：在jvm.options中设置 -Xms和-Xmx：

1. 两个值设置为相等

2. 将 -Xmx 设置为不超过物理内存的一半

   > 有原因的，可见 atGuiGu 的笔记

#### 4.1.3 配置文件3：log4j2.properties

日志文件设置，ES 使用 log4j，注意日志级别的配置。

#### 4.1.4 正式使用

**启动**：启动Elasticsearch：`bin\elasticsearch.bat`，es 的特点就是开箱即，无需配置，启动即可。

注意：es7 windows版本不支持机器学习，所以 elasticsearch.yml 中添加如下几个参数：

```yaml
node.name: node-1  
cluster.initial_master_nodes: ["node-1"]  
# 关闭机器学习
xpack.ml.enabled: false 
# 跨站访问相关
http.cors.enabled: true
http.cors.allow-origin: /.*/
```

**检查ES是否启动成功**：浏览器访问http://localhost:9200/?pretty

```json
{
  "name" : "LAPTOP-F6OAPAAB",  # name: node名称，取自机器的hostname
  "cluster_name" : "elasticsearch",  # cluster_name: 集群名称（默认的集群名称就是elasticsearch）
  "cluster_uuid" : "8gvoNVYYRieneidrT40bXA",
  "version" : {
    "number" : "7.8.0",  # version.number: 7.3.0，es版本号
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "757314695644ea9a1dc2fecd26d1a43856725e65",
    "build_date" : "2020-06-14T19:35:50.234439Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",  # version.lucene_version:封装的lucene版本号
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

查询集群状态：浏览器访问 http://localhost:9200/_cluster/health 

```json
{
    "cluster_name":"elasticsearch",
    "status":"green",  # Status：集群状态。Green 所有分片可用。Yellow所有主分片可用。Red主分片不可用，集群不可用。
    "timed_out":false,
    "number_of_nodes":1,
    "number_of_data_nodes":1,
    "active_primary_shards":0,
    "active_shards":0,
    "relocating_shards":0,
    "initializing_shards":0,
    "unassigned_shards":0,
    "delayed_unassigned_shards":0,
    "number_of_pending_tasks":0,
    "number_of_in_flight_fetch":0,
    "task_max_waiting_in_queue_millis":0,
    "active_shards_percent_as_number":100
}
```

### 4.2 Windows安装Kibana

1. Kibana是ES数据的前端展现，数据分析时，可以方便地看到数据。作为开发人员，可以方便访问ES。
2. 下载，解压Kibana。
3. 启动Kibana：`bin\kibana.bat`
4. 浏览器访问 http://localhost:5601 进入Dev Tools界面。
5. 发送get请求，查看集群状态`GET _cluster/health`

**Dev Tools界面**：![DevTools界面](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415879.png)

**监控集群界面**：![集群监控界面](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415265.png)

**集群状态（搜索速率、索引速率等）**：![集群状态界面](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415636.png)

### 4.3 Windows安装Postman

Postman是一个模拟 http 请求的工具。能够非常细致地定制化各种http请求。如`get\post\put\delete`，携带body参数等。

在没有Kibana时，可以使用Postman调试。

使用：get请求访问http://localhost:9200/

![PostmanGet](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415026.png)                                             

测试一下get方式查询集群状态：http://localhost:9200/_cluster/health

![PostmanGet查询集群状态](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415210.png)

### 4.4 Windows安装head插件

head插件是 ES 的一个可视化管理插件，用来监视 ES 的状态，并通过 head 客户端和 ES 服务进行交互，比如创建映射、创建索引等，head 的项目地址在https://github.com/mobz/elasticsearch-head 。

从ES6.0开始，head插件支持使得node.js运行

**直接在Chrome上安装插件就行了**

**运行**：![Head插件](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415324.png)

> 打开浏览器调试工具发现报错：
>
> 原因是：head 插件作为客户端要连接ES服务（localhost:9200），此时存在跨域问题，elasticsearch默认不允许跨域访问。
>
> 设置 elasticsearch 允许跨域访问。
>
> 在 `config/elasticsearch.yml` 后面增加以下参数：
>
> ```properties
> # 开启cors跨域访问支持，默认为false   
> http.cors.enabled: true   
> # 跨域访问允许的域名地址，(允许所有域名)以上使用正则   
> http.cors.allow-origin: /.*/
> ```
>
> 注意：将config/elasticsearch.yml 另存为 UTF-8 编码格式。

成功连接ES：![head插件成功连接ES](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261415349.png)

> 注意：`kibana\postman\head`插件选择自己喜欢的一种使用即可。
>
> 本教程使用Kibana的dev tool，因为地址栏省略了http://localhost:9200。