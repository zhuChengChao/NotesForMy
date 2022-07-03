# 中间件：CAT学习笔记-1

> 实习过程中了解到存在调用链监控这种好用的工具，就想稍微系统的学习一下，发现也有相应的视频课程：[Java进阶教程Cat入门](https://www.bilibili.com/video/BV1m64y127f3)，就顺道看了一波，跟着课程内容做了下学习笔记，此为 1/3 篇。

## 0. 学习目标

- 能够知道什么是CAT
- 能够搭建CAT服务端环境
- 能够进行CAT客户端的集成
- 能够使用CAT监控界面进行服务监控
- 能够完成CAT和常用框架集成
- 了解CAT告警配置
- 了解CAT客户端和服务端原理

## 1. CAT入门

在这一部分我们主要介绍以下3部分内容：

- 什么是调用链监控

- 什么是CAT

- CAT报表介绍

### 1.1 什么是调用链监控

#### 1.1.1 架构的演进历史

**单体应用**

![单体应用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405859.png)

架构说明：全部功能集中在一个项目内（All in one）

在单体应用的年代，分析线上问题主要靠日志以及系统级别的指标

**微服务架构**

![微服务架构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405497.png)

架构说明：将系统服务层完全独立出来，抽取为一个一个的微服务

当我们开始微服务架构之后，**服务变成分布式**的了，并且对服务进行了拆分。当用户的一个请求进来，会依次经过不同的服务节点进行处理，处理完成后再返回结果给用户。那么在整个处理的链条中，如果有任何一个节点出现了延迟或者问题，都有可能导致最终的结果出现异常，有的时候不同的服务节点甚至是由不同的团队开发的、部署在不同的服务器上，那么在这么错综复杂的环境下，我们想要排查出是链条中的具体哪个服务节点出了问题，其实并不容易。如下图片很形象的解释了在微服务架构下的复杂调用关系：

![服务的调用关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405981.png)

#### 1.1.2 调用链监控的需求

**调用链监控**是在微服务架构中非常重要的一环。它除了能帮助我们定位问题以外，还能帮助项目成员清晰的去了解项目部署结构，毕竟一个几十上百的微服务，相信在运行时间久了之后，项目的结构会出现上述非常复杂的调用链，在这种情况下，团队开发者甚至是架构师都不一定能对项目的网络结构有很清晰的了解，那就更别谈系统优化了。

这里我们会使用到**调用链监控工具**，那么首先我们先对调用链监控工具提出我们的需求：

1. 线上的服务是否运行正常。是不是有一些服务已经宕机了，但是我们没有发现呢？如何快速发现已经宕机的服务？

2. 来自用户的一笔调用失败了，到底是哪个服务导致的错误，我们需要能够快速定位到才能做到修复。

3. 用户反映，我们的系统很“慢”。如何知道究竟慢在何处？

从上述问题可以看出，微服务架构下，如果没有一款强大的调用链监控工具，势必会产生如下问题：

- 问题处理不及时，影响用户的体验
- 不同应用的负责人不承认是自己的问题导致失败，容易出现“扯皮”
- 服务之间的调用关系难以梳理，可能会存在很多错误的调用关系
- 由于没有具体的数据，团队成员对自己的应用性能不在意

#### 1.1.3 调用链监控的原理

在2010年，google发表了一篇名为“Dapper, a Large-Scale Distributed Systems Tracing Infrastructure”的论文，在文中介绍了google生产环境中大规模分布式系统下的跟踪系统Dapper的设计和使用经验。而如今很多的调用链系统如 zipkin/pinpoint 等系统都是基于这篇文章而实现的。

接下来我们就简单的介绍一下Dapper中调用链监控的原理：

![dapper](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405524.png)

如上图所示，这是一个查询订单的简单业务，他有如下的步骤：

1. 前端浏览器发起请求到订单服务，订单服务会从数据库中查询出对应的订单数据。订单数据中包含了商品的ID，所以还需要查询商品信息。

2. 订单服务发起一笔调用，通过 RPC 的方式，远程调用商品服务的查询商品信息接口。

3. 订单服务组装数据，返回给前端。

这几个步骤中，有几个核心概念需要了解：

- **Trace**:

  Trace 是指**一次请求调用的链路过程**，Trace ID 是指这次请求调用的 ID。在一次请求中，会在网络的最开始生成一个全局唯一的用于标识此次请求的 Trace ID，这个 Trace ID 在这次请求调用过程中无论经过多少个节点都会保持不变，并且在随着每一层的调用不停的传递。最终，可以通过 Trace ID 将这一次用户请求在系统中的路径全部串起来。

- **Span**:

  Span 是指一个模块的调用过程，一般用 Span ID 来标识。在一次请求的过程中会调用不同的节点/模块/服务，每一次调用都会生成一个新的  Span ID 来记录。这样，就可以通过  Span ID 来定位当前请求在整个系统调用链中所处的位置，以及它的上下游节点分别是什么。

那么回到上面的案例中，查询订单数据和查询商品数据这两个过程，就分别是两个 Span，我们记为 Span A和B。B的parent也就是父Span就是A。这两个Span都拥有同一个Trace Id：1。

并且在信息收集过程中，会记录调用的开始时间，结束时间，从中计算出调用的耗时。

这样，就可以清楚的知道，每笔调用：

- 经过了哪几个服务以及服务的调用顺序
- 每个服务过程的耗时

### 1.2 什么是 CAT

#### CAT

![cat](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405901.png)

CAT 是由大众点评开源的一款调用链监控系统，基于 JAVA 开发的。有很多互联网企业在使用，热度非常高。它有一个**非常强大和丰富的可视化报表界面**，这一点其实对于一款调用链监控系统而来非常的重要。在 CAT 提供的报表界面中有非常多的功能，几乎能看到你想要的任何维度的报表数据。

**特点**：聚合报表丰富，中文支持好，国内案例多

国内案例：携程、点评、陆金所等

#### PinPoint

Pinpoint是由一个韩国团队实现并开源，针对 Java 编写的大规模分布式系统设计，通过 JavaAgent 的机制做字节代码植入，实现加入 Trace ID 和获取性能数据的目的，**对应用代码零侵入**。

特点：支持多种插件，UI 功能强大，接入端无代码侵入

官方网站：https://github.com/naver/pinpoint

![pinpoint](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405292.png)

#### SkyWalking

SkyWalking 是 apache 基金会下面的一个开源 APM 项目，为微服务架构和云原生架构系统设计。它通过探针自动收集所需的指标，并进行分布式追踪。通过这些调用链路以及指标，Skywalking APM 会感知应用间关系和服务间关系，并进行相应的指标统计。Skywalking 支持链路追踪和监控应用组件基本涵盖主流框架和容器，如国产 RPC Dubbo和motan等，国际化的spring boot，spring cloud。

特点：支持多种插件，UI功能较强，**接入端无代码侵入**

官方网站：http://skywalking.apache.org/

![skywalking](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405097.jpg)

#### Zipkin

Zipkin 是由 Twitter 开源，是分布式链路调用监控系统，聚合各业务系统调用延迟数据，达到链路调用监控跟踪。Zipkin 基于 Google 的 Dapper 论文实现，主要完成数据的收集、存储、搜索与界面展示。

特点：轻量，使用部署简单

官方网站：https://zipkin.io/

![zipkin](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405760.png)

### 1.3 CAT 报表介绍

**CAT 支持如下报表**：

| 报表名称         | 报表内容                                                     |
| ---------------- | ------------------------------------------------------------ |
| Transaction 报表 | 一段代码的运行时间、次数、比如 URL/cache/sql 执行次数相应时间 |
| Event 报表       | 一段代码运行次数，比如出现一次异常                           |
| Problem 报表     | 根据 Transaction/Event 数据分析出系统可能出现的一次慢程序    |
| Heartbeat 报表   | JVM 状态信息                                                 |
| Business 报表    | 业务指标等，用户可以自己定制                                 |

**Transaction 报表**：

![transaction_view](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405573.png)

![transction报表4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405844.png)

![transaction_chart1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405223.png)

**Event报表**：

![event](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405172.png)![event_name](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406745.png)

**Problem报表**：

![problem](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405596.png)**Heartbeat报表**：

![heartbeat_view](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405152.png)

**Business报表**：

![problem](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405739.png)

## 2. CAT基础

### 2.1 下载与安装

**模块介绍**：

- cat-client：客户端，上报监控数据

- cat-consumer：服务端，收集监控数据进行统计分析，构建丰富的统计报表

- cat-alarm：实时告警，提供报表指标的监控告警

- cat-hadoop：数据存储，logview 存储至 Hdfs

- cat-home：管理端，报表展示、配置管理等

### 2.2 客户端集成

#### 2.2.1 简单案例

接下来我们编写一个简单的 SpringBoot 与 Cat 整合的案例，首先创建一个 SpringBoot 的初始化工程。只需要添加 web 依赖即可

**添加 Maven 添加依赖**

> 这需要通过源码 install 到本地仓库中，通过源码安装巨坑！！！直接下载包安装好了：http://unidal.org/nexus/content/repositories/releases/com/dianping/cat/cat-client/3.0.0/cat-client-3.0.0.jar
>
> 运行命令：`mvn install:install-file -Dfile=cat-client-3.0.0.jar -DgroupId=com.dianping.cat -DartifactId=cat-client -Dversion=3.0.0 -Dpackaging=jar`

```xml
<dependency>
    <groupId>com.dianping.cat</groupId>
    <artifactId>cat-client</artifactId>
    <version>3.0.0</version>
</dependency>
```

**启动 cat 客户端前的准备工作**

以下所有文件，如果在windows下，需要创建在启动项目的盘符下（即和 Tomcat 同样的盘符）

1. 创建 `/data/appdatas/cat` 目录，确保你具有这个目录的读写权限。

2. 创建 `/data/applogs/cat` 目录 (可选)，这个目录是用于存放运行时日志的，这将会对调试提供很大帮助，同样需要读写权限。

3. 创建 `/data/appdatas/cat/client.xml`，内容如下

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <config xmlns:xsi="http://www.w3.org/2001/XMLSchema" xsi:noNamespaceSchemaLocation="config.xsd">
       <servers>
           <server ip="127.0.0.1" port="2280" http-port="8080" />
       </servers>
   </config>
   ```

**初始化**

在你项目中创建 `src/main/resources/META-INF/app.properties` 文件, 并添加如下内容:

```
app.name={appkey}
```

> appkey 只能包含英文字母 (a-z, A-Z)、数字 (0-9)、下划线 (_) 和中划线 (-)

![app初始化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405206.png)

**编写代码**

在 com.itcast.springbootcat 包下创建 CatController

```java
package com.itcast.springbootcat;

import com.dianping.cat.Cat;
import com.dianping.cat.message.Event;
import com.dianping.cat.message.Transaction;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CatController {

    @RequestMapping("test")
    public String test(){

        Transaction t = Cat.newTransaction("URL", "pageName");

        try {
            Cat.logEvent("URL.Server", "serverIp", Event.SUCCESS, "ip=${serverIp}");
            Cat.logMetricForCount("metric.key");
            Cat.logMetricForDuration("metric.key", 5);

            // 让代码抛出异常
            int i = 1/0;
            t.setStatus(Transaction.SUCCESS);
        } catch (Exception e) {
            t.setStatus(e);
            Cat.logError(e);
        } finally {
            t.complete();
        }

        return "hello cat";
    }
}

```

**运行SpringBoot**

> 由于 8080 端口已经被占用了，因此此处需要修改 SpringBoot 的端口信息

启动 SpringBoot 项目，访问接口 http://[ip:端口]/test。然后在 Cat 中查看结果。

![首次启动1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405826.png)

如上图所示，已经出现了一笔调用，我们来看下调用的细节。

![首次启动2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405246.png)

查看具体的错误信息：

![首次启动3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261405805.png)

很显然看出上图所示其实是一个除0异常，到此为止 SpringBoot 客户端集成 Cat 就完成了。

#### 2.2.2 API介绍

##### 2.2.2.1 Transaction

**Transaction：适合记录跨越系统边界的程序访问行为，比如远程调用、数据库调用，也适合执行时间较长的业务逻辑监控，Transaction 用来记录一段代码的执行时间和次数**

现在我们的框架还没有与 dubbo、mybatis 做集成，所以我们通过手动编写一个本地方法，来测试 Transaction 的用法，创建 TransactionController 用于测试。

```java
package com.itcast.springbootcat.api;

import com.dianping.cat.Cat;
import com.dianping.cat.message.Transaction;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/transaction")
public class TransactionController {

    @RequestMapping("/test")
    public String test(){
        // 开启第一个Transaction，类别为URL,名称为test
        Transaction t = Cat.newTransaction("URL", "test");

        try {
            dubbo();
            t.setStatus(Transaction.SUCCESS);
        } catch (Exception e) {
            t.setStatus(e);
            Cat.logError(e);
        } finally {
            t.complete();
        }
        return "test";
    }

    private String dubbo(){
        // 开启第二个Transaction，类别为DUBBO,名称为dubbo
        Transaction t = Cat.newTransaction("DUBBO", "dubbo");

        try {
            t.setStatus(Transaction.SUCCESS);
        } catch (Exception e) {
            t.setStatus(e);
            Cat.logError(e);
        } finally {
            t.complete();
        }

        return "test";
    }
}
```

上面的代码中，开启了两个 Transaction，其中第一个 Transaction 为 Controller 接收到的接口调用，第二个为我们编写的本地方法 dubbo 用来模拟远程调用，在方法内部，开启第二个 Transaction。

启动项目，访问接口http://localhost:9000/transaction/test

![两次调用1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406523.png)

点击左侧菜单 Transaction 报表，选中 URL 类型对应的 Log View 查看调用链关系

![两次调用2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406997.png)

如图所示调用链已经形成，可以看到类型为 URL 的 test 调用了类型为 DUBBO 的 dubbo 方法，分别耗时 0.02ms和 0.01ms。

**扩展API**：CAT 提供了一系列 API 来对 Transaction 进行修改。

- `addData`：添加额外的数据显示
- `setStatus`：设置状态，成功可以设置 SUCCESS，失败可以设置异常
- `setDurationInMillis`：设置执行耗时（毫秒）
- `setTimestamp`：设置执行时间
- `complete`：结束 Transaction 

编写如下代码进行测试：

```java
@RequestMapping("/api")
public String api(){
    Transaction t = Cat.newTransaction("URL", "pageName");

    try {
        // 设置执行时间1秒
        t.setDurationInMillis(1000);
        t.setTimestamp(System.currentTimeMillis());
        // 添加额外数据
        t.addData("content");
        // 设置状态，成功状态
        t.setStatus(Transaction.SUCCESS);
    } catch (Exception e) {
        // 设置状态，失败状态
        t.setStatus(e);
        Cat.logError(e);
    } finally {
        // 结束Transaction 
        t.complete();
    }
    return "api";
}
```

启动项目，访问接口http://localhost:9000/transaction/api，点击左侧菜单 Transaction 报表，选中 URL 类型对应的 Log View 查看调用链关系：

![扩展API](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406522.png)

如图所示，调用耗时已经被手动修改成了1000ms，并且添加了额外的信息 content

>在使用 Transaction API 时，你可能需要注意以下几点：
>
>1. 你可以调用 `addData` 多次，添加的数据会被 `&` 连接起来
>2. 不要忘记完成complete transaction！否则你会得到一个毁坏的消息树以及内存泄漏！

##### 2.2.2.2 Event

**Event 用来记录一件事发生的次数，比如记录系统异常，它和 Transaction 相比缺少了时间的统计，开销比 Transaction 要小**

**Cat.logEvent**

```java
// 记录一个事件 
Cat.logEvent("URL.Server", "serverIp", Event.SUCCESS, "ip=${serverIp}");

// public static void logEvent(String type, String name){}
// public static void logEvent(String type, String name, String status, String nameValuePairs){}
```

**Cat.logError**

记录一个带有错误堆栈信息的 Error

* `public static void logError(String message, Throwable cause){}`
* `public static void logError(Throwable cause){}`

Error 是一种特殊的事件，它的 `type` 取决于传入的 `Throwable e`

1. 如果 `e` 是一个 `Error`，`type` 会被设置为 `Error`
2. 如果 `e` 是一个 `RuntimeException`，`type` 会被设置为 `RuntimeException`。
3. 其他情况下，`type` 会被设置为 `Exception`。

同时错误堆栈信息会被收集并写入 `data` 属性中。

```java
try {
    int i = 1 / 0;
} catch (Throwable e) {
    Cat.logError(e);
}
```

你可以向错误堆栈顶部添加你自己的错误消息，如下代码所示：

```java
Cat.logError("error(X) := exception(X)", e);
```

编写案例测试上述API：

```java
@RestController
@RequestMapping("/event")
public class EventController {

    @RequestMapping("/logEvent")
    public String logEvent(){
        // 参数1：type类型   参数2：名称   参数3：状态   参数4：携带信息
        Cat.logEvent("URL.Server", "serverIp", Event.SUCCESS, "ip=127.0.0.1");
        return "test";
    }

    @RequestMapping("/logError")
    public String logError(){
        try {
            int i = 1 / 0;
        } catch (Throwable e) {
            Cat.logError("error(X) := exception(X)", e);
        }
        return "test";
    }
}
```

启动项目，访问接口http://localhost:9000/event/logEvent和http://localhost:9000/event/logError。

![event事件1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406597.png)

通过上图可以看到，增加了两个事件：URL.Server 和 RuntimeException，点开LOG查看。

![event事件](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406825.png)

这里出现了两个 Event 的详细内容：

1. URL.Server 是一个正常的事件，打印出了 IP=127.0.0.1 的信息

2. RuntimeException 是一个错误 Event，不仅打印出了错误堆栈，还将我们打印的 `error(X) := exception(X)`内容放到了堆栈的最上方便于查看。

##### 2.2.2.3 Metric

**Metric 用于记录业务指标、指标可能包含对一个指标记录次数、记录平均值、记录总和，业务指标最低统计粒度为1分钟**

```java
// Counter，累加
Cat.logMetricForCount("metric.key");  // 调用一次+1
Cat.logMetricForCount("metric.key", 3);  // 调用一次+3

// Duration，累加取平均	
Cat.logMetricForDuration("metric.key", 5);
```

我们**每秒**会聚合 metric 然后发送，举例来说，如果你在同一秒调用 count 三次（相同的 name），累加他们的值，并且一次性上报给服务端

在 `duration` 的情况下，用平均值来取代累加值。

编写案例测试上述API：

```java
@RestController
@RequestMapping("/metric")
public class MetricController {

    @RequestMapping("/count")
    public String count(){
        Cat.logMetricForCount("count");
        return "test";
    }

    @RequestMapping("/duration")
    public String duration(){
        Cat.logMetricForDuration("duration", 1000);
        return "test";
    }
}

```

启动项目，访问接口http://localhost:9000/metric/count 点击5次和 http://localhost:9000/metric/duration。

![metric结果](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406992.png)

通过上图可以看到，count和duration的具体数值。

count一共点击了5次，所以这**一分钟**内数值为5；而duration不管点击多少次，由于取的是平均值，所以一直是1000。

#### 2.2.3 CAT 监控界面介绍

##### 2.2.3.1 DashBoard

DashBoard 仪表盘显示了每分钟出现错误的系统及其错误的次数和时间

![DashBoard](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406068.png)

- 点击右上角的时间按钮可以切换不同的展示时间，-7d代表7天前，-1h代表1小时前，now定位到当前时间
- 上方的时间轴按照分钟进行排布，点击之后可以看到该时间到结束的异常情况
- 下方标识了出错的系统和出错的时间、次数，点击系统名称可以跳转到 Problem 报表

##### 2.2.3.2 Transaction

Transaction 报表用来监控一段代码运行情况：**运行次数、错误次数、失败率、响应时间统计（平均影响时间、Tp分位值）、QPS**等等

应用启动后默认会打点的部分：

| 打点   | 来源组件            | 描述                   |
| ------ | ------------------- | ---------------------- |
| System | cat-client          | 上报监控数据的打点信息 |
| URL    | 需要接入 cat-filter | URL 访问的打点信息     |

**小时报表**

Type 统计界面展示了一个 Transaction 的第一层分类的视图，可以知道这段时间里面一个分类运行的次数，平均响应时间，延迟，以及分位线。

![transction报表1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406863.png)

从上而下分析报表：

1. **报表的时间跨度**：CAT默认是以一小时为统计时间跨度，点击【切到历史模式】，更改查看报表的时间跨度：默认是小时模式；切换为历史模式后，右侧快速导航，变为 month(月报表)、week(周报表)、day(天报表)，可以点击进行查看，注意报表的时间跨度会有所不同。

2. **时间选择**：通过右上角时间导航栏选择时间：点击 [+1h]/[-1h] 切换时间为下一小时/上一小时；点击 [+1d]/[-1d] 切换时间为后一天的同一小时/前一天的同一小时；点击右上角 [+7d]/[-7d] 切换时间为后一周的同一小时/前一周的同一小时；点击 [now] 回到当前小时。

3. **项目选择** 输入项目名，查看项目数据；如果需要切换其他项目数据，输入项目名，回车即可。

4. **机器分组** CAT可以将若干个机器，作为一个分组进行数据统计。默认会有一个 All 分组，代表所有机器的统计数据，即集群统计数据。

5. **所有Type汇总表格** 第一层分类（Type），点击查看第二级分类（称为 name）数据：

   ![transction报表2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406625.png)

   - Transaction 的埋点的 Type 和 Name 由业务自己定义，**当打点了 `Cat.newTransaction(type, name)` 时，第一层分类是type，第二级分类是name**。
   - 第二级分类数据是统计相同 type 下的所有 name 数据，数据均与第一级（type）一样的展示风格

6. **单个Type指标图表**：点击show，查看 Type 所有 name 分钟级统计，如下图: 

   ![transction报表3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406913.png)

7. **指标说明**：显示的是小时粒度第一级分类（type）的次数、错误数、失败率等数据。

8. **样本logview**：L代表logview，为一个样例的调用链路。

   ![transction报表4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406403.png)

9. **分位线说明**：小时粒度的时间第一级分类（type）相关统计

   - 95 line：表示95%的请求的响应时间比参考值要小；
   - 999 line：表示99.9%的响应时间比参考值要小；
   - 95line以及99line，也称之为tp95、tp99。

**历史报表**

Transaction 历史报表支持每天、每周、每月的数据统计以及趋势图，点击导航栏的【切换历史模式】进行查询。Transaction历史报表以：响应时间、访问量、错误量三个维度进行展示，以天报表为例：选取一个type，点击show，即可查看天报表。

##### 2.2.3.3 Event

Event 报表监控一段代码运行次数：例如记录程序中一个事件记录了多少次，错误了多少次。**Event报表的整体结构与Transaction报表几乎一样，只缺少响应时间的统计。**

**Type：第一级分类（Type）统计界面**

Type 统计界面展示了一个 Event 的第一层分类的视图，Event 相对于 Transaction 少了运行时间统计。可以知道这段时间里面一个分类运行的次数，失败次数，失败率，采样 logView，QPS。

![Event1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406044.png)

**Name：第二级分类（Name）统计界面**

第二级分类在 Type 统计界面中点击具体的 Type 进入，展示的是相同 Type 下所有的 Name 数据，可以理解为某 Type 下更细化的分类。

![event2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406344.png)

##### 2.2.3.4 Problem

Problem 记录整个项目在运行过程中出现的问题，包括一些**异常、错误、访问较长的行为**。Problem 报表是由 logview 存在的特征整合而成，方便用户定位问题。 

**来源：**

1. 业务代码**显示调用 `Cat.logError(e)` API 进行埋点**，具体埋点说明可查看埋点文档。
2. 与 **LOG 框架集成**，会捕获 log 日志中有异常堆栈的exception日志。
3. long-url，表示 Transaction 打点 URL 的慢请求，即 Type 类型要设置为long-url，后同
4. long-sql，表示 Transaction 打点 SQL 的慢请求
5. long-service，表示 Transaction 打点 Service 或者 PigeonService 的慢请求
6. long-call，表示 Transaction 打点 Call 或者 PigeonCall 的慢请求
7. long-cache，表示 Transaction 打点 Cache. 开头的慢请求

**所有错误汇总报表**：

第一层分类（Type），代表错误类型，比如 error、long-url、heartbeat 等；

第二级分类（称为 Status），对应具体的错误，比如一个异常类名等

![problem报表1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406462.png)

**错误数分布** 点击 Type 和 Status 的 ::show::，分别展示 Type 和 Status 的分钟级错误数分布：

![problem报表2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406665.png)

##### 2.2.3.5 HeartBeat

Heartbeat 报表是 CAT 客户端，以一分钟为周期，定期向服务端汇报当前运行时候的一些状态

**JVM相关指标**：以下所有的指标统计都是1分钟内的值，CAT 最低统计粒度是一分钟

| JVM GC 相关指标                 | 描述                     |
| ------------------------------- | ------------------------ |
| NewGC Count / PS Scavenge Count | 新生代GC次数             |
| NewGC Time / PS Scavenge Time   | 新生代GC耗时             |
| OldGC Count                     | 老年代GC次数             |
| PS MarkSweepTime                | 老年代GC耗时             |
| Heap Usage                      | Java虚拟机堆的使用情况   |
| None Heap Usage                 | Java虚拟机Perm的使用情况 |

![heartbeat报表1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406131.png)

| JVM Thread 相关指标  | 描述                    |
| -------------------- | ----------------------- |
| Active Thread        | 系统当前活动线程        |
| Daemon Thread        | 系统后台线程            |
| Total Started Thread | 系统总共开启线程        |
| Started Thread       | 系统每分钟新启动的线程  |
| CAT Started Thread   | 系统中CAT客户端启动线程 |

![heartbeat报表2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406181.png)

> 可以参考java.lang.management.ThreadInfo的定义

**系统指标**

| System 相关指标     | 描述                 |
| ------------------- | -------------------- |
| System Load Average | 系统 Load 详细信息   |
| Memory Free         | 系统 memoryFree 情况 |
| FreePhysicalMemory  | 物理内存剩余空间     |
| / Free              | /根的使用情况        |
| /data Free          | /data盘的使用情况    |

![heartbeat报表3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406312.png)

![heartbeat报表4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406500.png)

##### 2.2.3.6 Business

Business 报表对应着业务指标，比如订单指标。与 Transaction、Event、Problem 不同，**Business 更偏向于宏观上的指标**，另外三者偏向于微观代码的执行情况。

场景示例：

1. 我想监控订单数量
2. 我想监控订单耗时

![Business报表1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406050.png)

**基线**：基线是对业务指标的预测值。

**基线生成算法：**最近一个月的4个每周几的数据加权求和平均计算得出，秉着更加信任新数据的原则，CAT 会基于历史数据做异常点的修正，会把一些明显高于以及低于平均值的点剔除。

举例：今天是2018-10-25（周四），今天整天基线数据的算法是最近四个周四（2018-10-18，2018-10-11，2018-10-04，2018-09-27）的每个分钟数据的加权求和或平均，权重值依次为1，2，3，4。如：当前时间为19:56分设为value，前四周对应的19:56分数据（由远及近）分别为 A,B,C,D，则 value = (A+2B+3C+4D) / 10。

对于刚上线的应用，第一天没有基线，第二天的基线基线是前一天的数据，以此类推。

**如何开启基线：**只有配置了基线告警的指标，才会自动计算基线。如需基线功能，请配置基线告警。

**注意事项：**

1. 打点尽量用纯英文，不要带一些特殊符号，例如 空格( )、分号(:)、竖线(|)、斜线(/)、逗号(,)、与号(&)、星号(*)、左右尖括号(<>)、以及一些奇奇怪怪的字符
2. 如果有分隔需求，建议用下划线(_)、中划线(-)、英文点号(.)等
3. 由于数据库不区分大小写，请尽量统一大小写，并且不要对大小写进行改动
4. 有可能出现小数：趋势图每个点都代表一分钟的值。假设监控区间是10分钟，且10分钟内总共上报5次，趋势图中该点的值为5%10=0.5

##### 2.2.3.7 State

State 报表显示了与CAT相关的信息

![state报表](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261406928.png)
