# 中间件：ELK学习笔记-7

> 很早之前粗略的学习过Lucene和Elasticsearch，然后最近实习过程中需要用到一些ES的基础操作，虽说使用起来没啥问题，但还是想稍微系统的学习一下，也能在秋招的时候稍微说道说道。
>
> 在 bilibili 上搜了一个课程，课程地址：[黑马：ELK高级搜索](https://www.bilibili.com/video/BV1Nt4y1m7qL)，跟着视频也做了些笔记，总的分了7篇，此为7/7篇。
>
> > 学的很草率...空了再来完善:cry:

## 20. Logstash学习

### 20.1 Logstash概述

![Logstash](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261417964.png)

**什么是 Logstash**

Logstash 是一个数据抽取工具，将数据从一个地方转移到另一个地方，如 hadoop 生态圈的 sqoop 等。

下载地址：https://www.elastic.co/cn/downloads/logstash

Logstash 之所以功能强大和流行，还与其丰富的过滤器插件是分不开的，过滤器提供的并不单单是过滤的功能，还可以对进入过滤器的原始数据进行复杂的逻辑处理，甚至添加独特的事件到后续流程中。

Logstash 配置文件有如下三部分组成，其中 input、output 部分是必须配置，filter 部分是可选配置，而 filter 就是过滤器插件，可以在这部分实现各种日志过滤功能。

**配置文件**

配置文件路径：`/config/xxx.conf`

```json 
input {
    # 输入插件
}
filter {
    # 过滤匹配插件
}
output {
    # 输出插件
}
```

**启动操作**

输入/输出：控制台

```bash
logstash.bat -e 'input{stdin{}} output{stdout{}}'
```

为了好维护，将配置写入文件，启动

```bash
logstash.bat -f ../config/test1.conf

# 文件test1.conf内容如下：
input{
	stdin{
	
	}
}

filter{

}

output{
	stdout{
		codec=>rubydebug
	}
}
```

启动成功后在控制台输入：

```bash
hello world
{
    "@timestamp" => 2021-08-15T08:51:51.843Z,
       "message" => "hello world\r",
      "@version" => "1",
          "host" => "LAPTOP-F6OAPAAB"
}
```

### 20.2 Logstash输入插件

https://www.elastic.co/guide/en/logstash/current/input-plugins.html

**标准输入Stdin**：从控制台中获取数据输出到控制台

```json
input{
    stdin{
       
    }
}
output {
    stdout{
        codec=>rubydebug    
    }
}
```

**读取文件 File**：logstash 使用一个名为 filewatch 的 ruby gem 库来监听文件变化，并通过一个叫 .sincedb 的数据库文件来记录被监听的日志文件的读取进度（时间戳），这个 sincedb 数据文件的默认路径在 `<path.data>/plugins/inputs/file`下面，文件名类似于 .sincedb_123456，而 `<path.data>` 表示 logstash 插件存储目录，默认是 LOGSTASH_HOME/data。

```json
input {
    file {
        path => ["/var/*/*"]
        start_position => "beginning"
    }
}
output {
    stdout{
        codec=>rubydebug    
    }
}
```

默认情况下，logstash 会从文件的结束位置开始读取数据，也就是说 logstash 进程会以类似 tail -f 命令的形式逐行获取数据。

**读取 TCP 网络数据：**

```json
input {
    tcp {
        port => "1234"
    }
}

filter {
    grok {
        match => { "message" => "%{SYSLOGLINE}" }
    }
}

output {
    stdout{
        codec=>rubydebug
    }
}
```

### 20.3 Logstash过滤器插件

https://www.elastic.co/guide/en/logstash/current/filter-plugins.html

#### 20.3.1 Grok 正则捕获

grok 是一个十分强大的 logstash filter 插件，他可以通过**正则解析任意文本**，将非结构化日志数据弄成结构化和方便查询的结构，他是目前 logstash 中解析非结构化日志数据最好的方式。

**Grok 的语法规则是**：`%{语法: 语义}`

例如输入的内容为：

```
172.16.213.132 [07/Feb/2019:16:24:19 +0800] "GET / HTTP/1.1" 403 5039
```

* `%{IP:clientip}` 匹配模式将获得的结果为 ：`clientip: 172.16.213.132`
* `%{HTTPDATE:timestamp}` 匹配模式将获得的结果为：`timestamp: 07/Feb/2018:16:24:19 +0800`
* `%{QS:referrer}`匹配模式将获得的结果为：`referrer: "GET / HTTP/1.1"`

下面是一个组合匹配模式，它可以获取上面输入的所有内容：

> `\空格`，为转义空格操作

```
%{IP:clientip}\ \[%{HTTPDATE:timestamp}\]\ %{QS:referrer}\ %{NUMBER:response}\ %{NUMBER:bytes}
```

通过上面这个组合匹配模式，我们将输入的内容分成了五个部分，即五个字段，将输入内容分割为不同的数据字段，这对于日后解析和查询日志数据非常有用，这正是使用 grok 的目的。

例子：

```json
input{
    stdin{}
}
filter{
    grok{
        match => ["message","%{IP:clientip}\ \[%{HTTPDATE:timestamp}\]\ %{QS:referrer}\ %{NUMBER:response}\ %{NUMBER:bytes}"]
    }
}
output{
    stdout{
        codec => "rubydebug"
    }
}
```


输入内容：

```json
172.16.213.132 [07/Feb/2019:16:24:19 +0800] "GET / HTTP/1.1" 403 5039
```

输出内容：

```json
{
      "clientip" => "172.16.213.132",
      "referrer" => "\"GET / HTTP/1.1\"",
          "host" => "LAPTOP-F6OAPAAB",
      "@version" => "1",
      "response" => "403",
       "message" => "}172.16.213.132 [07/Feb/2019:16:24:19 +0800] \"GET / HTTP/1.1\" 403 5039\r",
     "timestamp" => "07/Feb/2019:16:24:19 +0800",
    "@timestamp" => 2021-08-15T09:14:15.152Z,
         "bytes" => "5039"
}
```

#### 20.3.2 时间处理 Date

date 插件是对于排序事件和回填旧数据尤其重要，它可以用来**转换日志记录中的时间字段**，变成 `LogStash::Timestamp` 对象，然后转存到 `@timestamp` 字段里，这在之前已经做过简单的介绍。

下面是 date 插件的一个配置示例（这里仅仅列出 filter 部分）：

```json
filter {
    grok {
        match => ["message", "%{HTTPDATE:timestamp}"]
    }
    date {
        match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
    }
}
```

#### 20.3.3 数据修改 Mutate

**正则表达式替换匹配字段**

gsub 可以通过正则表达式替换字段中匹配到的值，只对字符串字段有效，下面是一个关于 mutate 插件中 gsub 的示例（仅列出 filter 部分）：

```json
filter {
    mutate {
        gsub => ["filed_name_1", "/" , "_"]
    }
}
```

这个示例表示将 filed_name_1 字段中所有`"/"`字符替换为`"_"`。

**分隔符分割字符串为数组**

split 可以通过指定的分隔符分割字段中的字符串为数组，下面是一个关于 mutate 插件中 split 的示例（仅列出filter部分）：

```json
filter {
    mutate {
        split => ["filed_name_2", "|"]
    }
}
```

这个示例表示将 filed_name_2 字段以`"|"`为区间分隔为数组。

**重命名字段**

rename 可以实现重命名某个字段的功能，下面是一个关于 mutate 插件中 rename 的示例（仅列出filter部分）：

```json
filter {
    mutate {
        rename => { "old_field" => "new_field" }
    }
}
```

这个示例表示将字段 old_field 重命名为 new_field。

**删除字段**

remove_field 可以实现删除某个字段的功能，下面是一个关于 mutate 插件中 remove_field 的示例（仅列出filter部分）：

```json
filter {
    mutate {
        remove_field  =>  ["timestamp"]
    }
}
```

这个示例表示将字段 timestamp 删除。

**GeoIP 地址查询归类**

```json
filter {
    geoip {
        source => "ip_field"
    }
}
```

**综合例子**

* filter设置：

```json
input {
	stdin {}
}

filter {
	grok {
    	match => { "message" => "%{IP:clientip}\ \[%{HTTPDATE:timestamp}\]\ %{QS:referrer}\ %{NUMBER:response}\ %{NUMBER:bytes}" }
        remove_field => [ "message" ]   # 删除了message字段
	}
    date {
		match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]  # 日期格式转换
	}
	mutate {
    	convert => [ "response","float" ]  # response数据格式转换
        rename => { "response" => "response_new" }  # response重命名
        gsub => ["referrer","\"",""]  # 替换内容，\替换为空
        split => ["clientip", "."]  # 对clientip分割
	}
}

output {
	stdout {
	    codec => "rubydebug"
	}
}
```

* 输入输出：

```json
172.16.213.132 [07/Feb/2019:16:24:19 +0800] "GET / HTTP/1.1" 403 5039

{
    "response_new" => "403",
        "referrer" => "GET / HTTP/1.1",
           "bytes" => "5039",
            "host" => "LAPTOP-F6OAPAAB",
      "@timestamp" => 2019-02-07T08:24:19.000Z,
        "clientip" => [
        [0] "172",
        [1] "16",
        [2] "213",
        [3] "132"
    ],
       "timestamp" => "07/Feb/2019:16:24:19 +0800",
        "@version" => "1"
}
```

### 20.4 Logstash输出插件

https://www.elastic.co/guide/en/logstash/current/output-plugins.html

output 是 Logstash 的最后阶段，一个事件可以经过多个输出，而一旦所有输出处理完成，整个事件就执行完成。

一些常用的输出包括：

- file：表示将日志数据写入磁盘上的文件。
- elasticsearch：表示将日志数据发送给 Elasticsearch，Elasticsearch 可以高效方便和易于查询的保存数据。

**输出到标准输出(stdout)**

```json
output {
    stdout {
        codec => rubydebug
    }
}
```

**保存为文件（file）**

```json
output {
    file {
        path => "/data/log/%{+yyyy-MM-dd}/%{host}_%{+HH}.log"
    }
}
```

**输出到elasticsearch**

```json
output {
    elasticsearch {
        host => ["192.168.1.1:9200","172.16.213.77:9200"]
        index => "logstash-%{+YYYY.MM.dd}"       
    }
}
```

参数设置：

- host：是一个数组类型的值，后面跟的值是 elasticsearch 节点的地址与端口，默认端口是 9200。可添加多个地址。
- index：写入 elasticsearch 的索引的名称，这里可以使用变量。Logstash提供了 %{+YYYY.MM.dd} 这种写法。在语法解析的时候，看到以 + 号开头的，就会自动认为后面是时间格式，尝试用时间格式来解析后续字符串。这种以天为单位分割的写法，可以很容易的删除老的数据或者搜索指定时间范围内的数据。此外，注意索引名中不能有大写字母。
- manage_template：用来设置是否开启 logstash 自动管理模板功能，如果设置为 false 将关闭自动管理模板功能。如果我们自定义了模板，那么应该设置为 false。
- template_name：这个配置项用来设置在 Elasticsearch 中模板的名称。

### 20.5 综合案例

```json
input {
    file {
        path => ["D:/ES/logstash-7.3.0/nginx.log"]        
        start_position => "beginning"
    }
}

filter {
	grok {
	     match => { "message" => "%{IP:clientip}\ \[%{HTTPDATE:timestamp}\]\ %{QS:referrer}\ %{NUMBER:response}\ %{NUMBER:bytes}" }
	     remove_field => [ "message" ]
	}
	date {
	    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
	}
	mutate {
	   rename => { "response" => "response_new" }
	   convert => [ "response","float" ]
	   gsub => ["referrer","\"",""]
	   remove_field => ["timestamp"]
	   split => ["clientip", "."]
	}
}

output {
    stdout {
        codec => "rubydebug"
    }
    elasticsearch {
        host => ["localhost:9200"]
        index => "logstash-%{+YYYY.MM.dd}"       
    }
}
```

## 21. 集群部署

**结点的三个角色**

* **主结点**：master 节点主要用于集群的管理及索引 比如新增结点、分片分配、索引的新增和删除等。
* **数据结点**：data 节点上保存了数据分片，它负责索引和搜索操作。 
* **客户端结点**：client 节点仅作为请求客户端存在，client 的作用也作为负载均衡器，client 节点不存数据，只是将请求均衡转发到其它结点。

通过下边两项参数来配置结点的功能：

```properties
node.master: # 是否允许为主结点
node.data:   # 允许存储数据作为数据结点
node.ingest: # 是否允许成为协调节点
```

四种组合方式：

```properties
master=true，data=true：   # 即是主结点又是数据结点
master=false，data=true：  # 仅是数据结点
master=true，data=false：  # 仅是主结点，不存储数据
master=false，data=false： # 即不是主结点也不是数据结点，此时可设置ingest为true表示它是一个客户端。
```

## 22 项目实战

### 22.1 项目一：ELK用于日志分析

**需求：集中收集分布式服务的日志**

1. 逻辑模块程序随时输出日志

```java
package com.itheima.es;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Random;

/**
 * creste by itheima.itcast
 */
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestLog {
    private static final Logger LOGGER= LoggerFactory.getLogger(TestLog.class);

    @Test
    public void testLog(){
        Random random =new Random();

        while (true){
            int userid=random.nextInt(10);
            LOGGER.info("userId:{},send:{}",userid,"hello world.I am "+userid);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

2. 日志配置文件

```xml
日志的配置文件：logback-spring.xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>
    <!--定义日志文件的存储地址,使用绝对路径-->
    <property name="LOG_HOME" value="./logs"/>

    <!-- Console 输出设置 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <!-- 按照每天生成日志文件 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <fileNamePattern>${LOG_HOME}/log-%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 异步输出 -->
    <appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>512</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref="FILE"/>
    </appender>


    <logger name="org.apache.ibatis.cache.decorators.LoggingCache" level="DEBUG" additivity="false">
        <appender-ref ref="CONSOLE"/>
    </logger>
    <logger name="org.springframework.boot" level="DEBUG"/>
    <root level="info">
        <!--<appender-ref ref="ASYNC"/>-->
        <appender-ref ref="FILE"/>
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

3. logstash 收集日志到 esv，需要写logstash配置文件。

```json
input {
    file {
        path => ["C:/Users/ZhuCC/Desktop/Elasticsearch/Code/es-itcast-demo/logs/log-*.log"]        
        start_position => "beginning"
    }
}

filter {
	grok {
	    match => { "message" => "%{DATA:datetime}\ \[%{DATA:thread}\]\ %{DATA:level}\ \ %{DATA:class} - %{GREEDYDATA:logger}" }
	    remove_field => [ "message" ]
	}
	date {
	    match => ["datetime", "yyyy-MM-dd HH:mm:ss.SSS"]
	}
	if "_grokparsefailure" in [tags]{
		drop { 
		}
	}
}

output {
    stdout {
        codec => "rubydebug"
    }
    elasticsearch {
        hosts => ["localhost:9200"]
        index => "logstash-%{+YYYY.MM.dd}"       
    }
}
```

> **grok 内置类型**
>
> ```properties
> USERNAME [a-zA-Z0-9._-]+
> USER %{USERNAME}
> INT (?:[+-]?(?:[0-9]+))
> BASE10NUM (?<![0-9.+-])(?>[+-]?(?:(?:[0-9]+(?:\.[0-9]+)?)|(?:\.[0-9]+)))
> NUMBER (?:%{BASE10NUM})
> BASE16NUM (?<![0-9A-Fa-f])(?:[+-]?(?:0x)?(?:[0-9A-Fa-f]+))
> BASE16FLOAT \b(?<![0-9A-Fa-f.])(?:[+-]?(?:0x)?(?:(?:[0-9A-Fa-f]+(?:\.[0-9A-Fa-f]*)?)|(?:\.[0-9A-Fa-f]+)))\b
> 
> POSINT \b(?:[1-9][0-9]*)\b
> NONNEGINT \b(?:[0-9]+)\b
> WORD \b\w+\b
> NOTSPACE \S+
> SPACE \s*
> DATA .*? 
> GREEDYDATA .*
> QUOTEDSTRING (?>(?<!\\)(?>"(?>\\.|[^\\"]+)+"|""|(?>'(?>\\.|[^\\']+)+')|''|(?>`(?>\\.|[^\\`]+)+`)|``))
> UUID [A-Fa-f0-9]{8}-(?:[A-Fa-f0-9]{4}-){3}[A-Fa-f0-9]{12}
> 
> # Networking 网络的解析，MAC地址、IPV6地址...
> MAC (?:%{CISCOMAC}|%{WINDOWSMAC}|%{COMMONMAC})
> CISCOMAC (?:(?:[A-Fa-f0-9]{4}\.){2}[A-Fa-f0-9]{4})
> WINDOWSMAC (?:(?:[A-Fa-f0-9]{2}-){5}[A-Fa-f0-9]{2})
> COMMONMAC (?:(?:[A-Fa-f0-9]{2}:){5}[A-Fa-f0-9]{2})
> IPV6 ((([0-9A-Fa-f]{1,4}:){7}([0-9A-Fa-f]{1,4}|:))|(([0-9A-Fa-f]{1,4}:){6}(:[0-9A-Fa-f]{1,4}|((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){5}(((:[0-9A-Fa-f]{1,4}){1,2})|:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){4}(((:[0-9A-Fa-f]{1,4}){1,3})|((:[0-9A-Fa-f]{1,4})?:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){3}(((:[0-9A-Fa-f]{1,4}){1,4})|((:[0-9A-Fa-f]{1,4}){0,2}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){2}(((:[0-9A-Fa-f]{1,4}){1,5})|((:[0-9A-Fa-f]{1,4}){0,3}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){1}(((:[0-9A-Fa-f]{1,4}){1,6})|((:[0-9A-Fa-f]{1,4}){0,4}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(:(((:[0-9A-Fa-f]{1,4}){1,7})|((:[0-9A-Fa-f]{1,4}){0,5}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:)))(%.+)?
> IPV4 (?<![0-9])(?:(?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2}))(?![0-9])
> IP (?:%{IPV6}|%{IPV4})
> HOSTNAME \b(?:[0-9A-Za-z][0-9A-Za-z-]{0,62})(?:\.(?:[0-9A-Za-z][0-9A-Za-z-]{0,62}))*(\.?|\b)
> HOST %{HOSTNAME}
> IPORHOST (?:%{HOSTNAME}|%{IP})
> HOSTPORT %{IPORHOST}:%{POSINT}
> 
> # paths
> PATH (?:%{UNIXPATH}|%{WINPATH})
> UNIXPATH (?>/(?>[\w_%!$@:.,-]+|\\.)*)+
> TTY (?:/dev/(pts|tty([pq])?)(\w+)?/?(?:[0-9]+))
> WINPATH (?>[A-Za-z]+:|\\)(?:\\[^\\?*]*)+
> URIPROTO [A-Za-z]+(\+[A-Za-z+]+)?
> URIHOST %{IPORHOST}(?::%{POSINT:port})?
> # uripath comes loosely from RFC1738, but mostly from what Firefox
> # doesn't turn into %XX
> URIPATH (?:/[A-Za-z0-9$.+!*'(){},~:;=@#%_\-]*)+
> #URIPARAM \?(?:[A-Za-z0-9]+(?:=(?:[^&]*))?(?:&(?:[A-Za-z0-9]+(?:=(?:[^&]*))?)?)*)?
> URIPARAM \?[A-Za-z0-9$.+!*'|(){},~@#%&/=:;_?\-\[\]]*
> URIPATHPARAM %{URIPATH}(?:%{URIPARAM})?
> URI %{URIPROTO}://(?:%{USER}(?::[^@]*)?@)?(?:%{URIHOST})?(?:%{URIPATHPARAM})?
> 
> # Months: January, Feb, 3, 03, 12, December
> MONTH \b(?:Jan(?:uary)?|Feb(?:ruary)?|Mar(?:ch)?|Apr(?:il)?|May|Jun(?:e)?|Jul(?:y)?|Aug(?:ust)?|Sep(?:tember)?|Oct(?:ober)?|Nov(?:ember)?|Dec(?:ember)?)\b
> MONTHNUM (?:0?[1-9]|1[0-2])
> MONTHNUM2 (?:0[1-9]|1[0-2])
> MONTHDAY (?:(?:0[1-9])|(?:[12][0-9])|(?:3[01])|[1-9])
> 
> # Days: Monday, Tue, Thu, etc...
> DAY (?:Mon(?:day)?|Tue(?:sday)?|Wed(?:nesday)?|Thu(?:rsday)?|Fri(?:day)?|Sat(?:urday)?|Sun(?:day)?)
> 
> # Years?
> YEAR (?>\d\d){1,2}
> HOUR (?:2[0123]|[01]?[0-9])
> MINUTE (?:[0-5][0-9])
> # '60' is a leap second in most time standards and thus is valid.
> SECOND (?:(?:[0-5]?[0-9]|60)(?:[:.,][0-9]+)?)
> TIME (?!<[0-9])%{HOUR}:%{MINUTE}(?::%{SECOND})(?![0-9])
> # datestamp is YYYY/MM/DD-HH:MM:SS.UUUU (or something like it)
> DATE_US %{MONTHNUM}[/-]%{MONTHDAY}[/-]%{YEAR}
> DATE_EU %{MONTHDAY}[./-]%{MONTHNUM}[./-]%{YEAR}
> ISO8601_TIMEZONE (?:Z|[+-]%{HOUR}(?::?%{MINUTE}))
> ISO8601_SECOND (?:%{SECOND}|60)
> TIMESTAMP_ISO8601 %{YEAR}-%{MONTHNUM}-%{MONTHDAY}[T ]%{HOUR}:?%{MINUTE}(?::?%{SECOND})?%{ISO8601_TIMEZONE}?
> DATE %{DATE_US}|%{DATE_EU}
> DATESTAMP %{DATE}[- ]%{TIME}
> TZ (?:[PMCE][SD]T|UTC)
> DATESTAMP_RFC822 %{DAY} %{MONTH} %{MONTHDAY} %{YEAR} %{TIME} %{TZ}
> DATESTAMP_RFC2822 %{DAY}, %{MONTHDAY} %{MONTH} %{YEAR} %{TIME} %{ISO8601_TIMEZONE}
> DATESTAMP_OTHER %{DAY} %{MONTH} %{MONTHDAY} %{TIME} %{TZ} %{YEAR}
> DATESTAMP_EVENTLOG %{YEAR}%{MONTHNUM2}%{MONTHDAY}%{HOUR}%{MINUTE}%{SECOND}
> 
> # Syslog Dates: Month Day HH:MM:SS
> SYSLOGTIMESTAMP %{MONTH} +%{MONTHDAY} %{TIME}
> PROG (?:[\w._/%-]+)
> SYSLOGPROG %{PROG:program}(?:\[%{POSINT:pid}\])?
> SYSLOGHOST %{IPORHOST}
> SYSLOGFACILITY <%{NONNEGINT:facility}.%{NONNEGINT:priority}>
> HTTPDATE %{MONTHDAY}/%{MONTH}/%{YEAR}:%{TIME} %{INT}
> 
> # Shortcuts
> QS %{QUOTEDSTRING}
> 
> # Log formats
> SYSLOGBASE %{SYSLOGTIMESTAMP:timestamp} (?:%{SYSLOGFACILITY} )?%{SYSLOGHOST:logsource} %{SYSLOGPROG}:
> COMMONAPACHELOG %{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-)
> COMBINEDAPACHELOG %{COMMONAPACHELOG} %{QS:referrer} %{QS:agent}
> 
> # Log Levels
> LOGLEVEL ([Aa]lert|ALERT|[Tt]race|TRACE|[Dd]ebug|DEBUG|[Nn]otice|NOTICE|[Ii]nfo|INFO|[Ww]arn?(?:ing)?|WARN?(?:ING)?|[Ee]rr?(?:or)?|ERR?(?:OR)?|[Cc]rit?(?:ical)?|CRIT?(?:ICAL)?|[Ff]atal|FATAL|[Ss]evere|SEVERE|EMERG(?:ENCY)?|[Ee]merg(?:ency)?)
> ```

4. kibana展现数据：通过 Discover 等组件查看数据

### 22.2 项目二：学成在线站内搜索模块

**步骤1：MySQL导入course_pub表**(表在资料中)

**步骤2：创建索引**

**步骤3：创建映射**

```json
PUT /xc_course
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "description" : {
            "analyzer" : "ik_max_word",
            "search_analyzer": "ik_smart",
            "type" : "text"
      },
      "grade" : {
         "type" : "keyword"
      },
      "id" : {
         "type" : "keyword"
      },
      "mt" : {
         "type" : "keyword"
      },
      "name" : {
      	"analyzer" : "ik_max_word",
     	"search_analyzer": "ik_smart",
          "type" : "text"
      },
      "users" : {
         "index" : false,
         "type" : "text"
      },
      "charge" : {
         "type" : "keyword"
      },
      "valid" : {
         "type" : "keyword"
      },
      "pic" : {
         "index" : false,
         "type" : "keyword"
      },
      "qq" : {
         "index" : false,
         "type" : "keyword"
      },
      "price" : {
         "type" : "float"
      },
      "price_old" : {
         "type" : "float"
      },
      "st" : {
         "type" : "keyword"
      },
      "status" : {
         "type" : "keyword"
      },
      "studymodel" : {
         "type" : "keyword"
      },
      "teachmode" : {
         "type" : "keyword"
      },
      "teachplan" : {
          "analyzer" : "ik_max_word",
     		"search_analyzer": "ik_smart",
          "type" : "text"
      },
     "expires" : {
         "type" : "date",
      	"format": "yyyy-MM-dd HH:mm:ss"
      },
      "pub_time" : {
         "type" : "date",
       	"format": "yyyy-MM-dd HH:mm:ss"
      },
      "start_time" : {
         "type" : "date",
     		"format": "yyyy-MM-dd HH:mm:ss"
      },
     "end_time" : {
          "type" : "date",
     		"format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  } 
}
```

**步骤4：logstash创建模板文件**

Logstash 的工作是从 MySQL 中读取数据，向 ES 中创建索引，这里需要提前创建 mapping 的模板文件以便logstash 使用。

在 logstach 的 config 目录创建 xc_course_template.json，内容如下：

```json
{
   "mappings" : {
      "doc" : {
         "properties" : {
            "charge" : {
               "type" : "keyword"
            },
            "description" : {
               "analyzer" : "ik_max_word",
               "search_analyzer" : "ik_smart",
               "type" : "text"
            },
            "end_time" : {
               "format" : "yyyy-MM-dd HH:mm:ss",
               "type" : "date"
            },
            "expires" : {
               "format" : "yyyy-MM-dd HH:mm:ss",
               "type" : "date"
            },
            "grade" : {
               "type" : "keyword"
            },
            "id" : {
               "type" : "keyword"
            },
            "mt" : {
               "type" : "keyword"
            },
            "name" : {
               "analyzer" : "ik_max_word",
               "search_analyzer" : "ik_smart",
               "type" : "text"
            },
            "pic" : {
               "index" : false,
               "type" : "keyword"
            },
            "price" : {
               "type" : "float"
            },
            "price_old" : {
               "type" : "float"
            },
            "pub_time" : {
               "format" : "yyyy-MM-dd HH:mm:ss",
               "type" : "date"
            },
            "qq" : {
               "index" : false,
               "type" : "keyword"
            },
            "st" : {
               "type" : "keyword"
            },
            "start_time" : {
               "format" : "yyyy-MM-dd HH:mm:ss",
               "type" : "date"
            },
            "status" : {
               "type" : "keyword"
            },
            "studymodel" : {
               "type" : "keyword"
            },
            "teachmode" : {
               "type" : "keyword"
            },
            "teachplan" : {
               "analyzer" : "ik_max_word",
               "search_analyzer" : "ik_smart",
               "type" : "text"
            },
            "users" : {
               "index" : false,
               "type" : "text"
            },
            "valid" : {
               "type" : "keyword"
            }
         }
      }
   },
   "template" : "xc_course"
}
```

**步骤5：logstash配置mysql.conf**

```json
input {
  stdin {
  }
  jdbc {
  	jdbc_connection_string => "jdbc:mysql://localhost:3306/xc_course？useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC"
  	# the user we wish to excute our statement as
  	jdbc_user => "root"
  	jdbc_password => root
  	# the path to our downloaded jdbc driver   数据库的驱动jar包
  	jdbc_driver_library => "D:/develop/maven/repository3/mysql/mysql-connector-java/5.1.41/mysql-connector-java-5.1.41.jar"
  	# the name of the driver class for mysql
  	jdbc_driver_class => "com.mysql.jdbc.Driver"
  	jdbc_paging_enabled => "true"
  	jdbc_page_size => "50000"
  	# 要执行的sql文件
  	# statement_filepath => "/conf/course.sql"
  	statement => "select * from course_pub where timestamp > date_add(:sql_last_value,INTERVAL 8 HOUR)"
  	# 定时配置
  	schedule => "* * * * *"
  	record_last_run => true
  	last_run_metadata_path => "D:/ES/logstash-7.3.0/config/logstash_metadata"
  }
}


output {
  elasticsearch {
  	# ES的ip地址和端口
  	hosts => "localhost:9200"
  	# hosts => ["localhost:9200"]
  	# ES索引库名称
  	index => "xc_course"
  	document_id => "%{id}"
  	document_type => "_doc"
  	template =>"D:/ES/logstash-7.3.0/config/xc_course_template.json"
  	template_name =>"xc_course"
  	template_overwrite =>"true"
  }
  stdout {
  	# 日志输出
  	codec => json_lines
  }
}
```

1. ES采用UTC时区问题

   ES采用UTC时区，比北京时间早8小时，所以ES读取数据时让最后更新时间加8小时

   `where timestamp > date_add(:sql_last_value,INTERVAL 8 HOUR)`

2. logstash每个执行完成会在/config/logstash_metadata记录执行时间下次以此时间为基准进行增量同步数据到索引库。

**步骤6：启动**

```bash
.\logstash.bat -f ..\config\mysql.conf
```

**步骤7：后端代码**

1. controller

```java
@RestController
@RequestMapping("/search/course")
public class EsCourseController  {
    @Autowired
    EsCourseService esCourseService;

    @GetMapping(value="/list/{page}/{size}")
    public QueryResponseResult<CoursePub> list(@PathVariable("page") int page, @PathVariable("size") int size, CourseSearchParam courseSearchParam) {
        return esCourseService.list(page,size,courseSearchParam);
    }
}
```

2. service

```java
@Service
public class EsCourseService {
    @Value("${heima.course.source_field}")
    private String source_field;

    @Autowired
    RestHighLevelClient restHighLevelClient;

    /**
     * 课程搜索
     */
    public QueryResponseResult<CoursePub> list(int page, int size, CourseSearchParam courseSearchParam) {
        if (courseSearchParam == null) {
            courseSearchParam = new CourseSearchParam();
        }
        // 1.创建搜索请求对象
        SearchRequest searchRequest = new SearchRequest("xc_course");

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        // 过虑源字段
        String[] source_field_array = source_field.split(",");
        searchSourceBuilder.fetchSource(source_field_array, new String[]{});
        // 创建布尔查询对象
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        // 搜索条件
        // 根据关键字搜索
        if (StringUtils.isNotEmpty(courseSearchParam.getKeyword())) {
            MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery(courseSearchParam.getKeyword(), "name", "description", "teachplan")
                .minimumShouldMatch("70%")
                .field("name", 10);
            boolQueryBuilder.must(multiMatchQueryBuilder);
        }
        if (StringUtils.isNotEmpty(courseSearchParam.getMt())) {
            // 根据一级分类
            boolQueryBuilder.filter(QueryBuilders.termQuery("mt", courseSearchParam.getMt()));
        }
        if (StringUtils.isNotEmpty(courseSearchParam.getSt())) {
            // 根据二级分类
            boolQueryBuilder.filter(QueryBuilders.termQuery("st", courseSearchParam.getSt()));
        }
        if (StringUtils.isNotEmpty(courseSearchParam.getGrade())) {
            // 根据难度等级
            boolQueryBuilder.filter(QueryBuilders.termQuery("grade", courseSearchParam.getGrade()));
        }

        // 设置boolQueryBuilder到searchSourceBuilder
        searchSourceBuilder.query(boolQueryBuilder);
        // 设置分页参数
        if (page <= 0) {
            page = 1;
        }
        if (size <= 0) {
            size = 12;
        }
        // 起始记录下标
        int from = (page - 1) * size;
        searchSourceBuilder.from(from);
        searchSourceBuilder.size(size);

        // 设置高亮
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.preTags("<font class='eslight'>");
        highlightBuilder.postTags("</font>");
        // 设置高亮字段
        // <font class='eslight'>node</font>学习
        highlightBuilder.fields().add(new HighlightBuilder.Field("name"));
        searchSourceBuilder.highlighter(highlightBuilder);

        searchRequest.source(searchSourceBuilder);

        QueryResult<CoursePub> queryResult = new QueryResult();
        List<CoursePub> list = new ArrayList<CoursePub>();
        try {
            // 2.执行搜索
            SearchResponse searchResponse = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
            // 3.获取响应结果
            SearchHits hits = searchResponse.getHits();
            long totalHits=hits.getTotalHits().value;
            // 匹配的总记录数
            // long totalHits = hits.totalHits;
            queryResult.setTotal(totalHits);
            SearchHit[] searchHits = hits.getHits();
            for (SearchHit hit : searchHits) {
                CoursePub coursePub = new CoursePub();
                // 源文档
                Map<String, Object> sourceAsMap = hit.getSourceAsMap();
                // 取出id
                String id = (String) sourceAsMap.get("id");
                coursePub.setId(id);
                // 取出name
                String name = (String) sourceAsMap.get("name");
                // 取出高亮字段name
                Map<String, HighlightField> highlightFields = hit.getHighlightFields();
                if (highlightFields != null) {
                    HighlightField highlightFieldName = highlightFields.get("name");
                    if (highlightFieldName != null) {
                        Text[] fragments = highlightFieldName.fragments();
                        StringBuffer stringBuffer = new StringBuffer();
                        for (Text text : fragments) {
                            stringBuffer.append(text);
                        }
                        name = stringBuffer.toString();
                    }
                }
                coursePub.setName(name);
                // 图片
                String pic = (String) sourceAsMap.get("pic");
                coursePub.setPic(pic);
                // 价格
                Double price = null;
                try {
                    if (sourceAsMap.get("price") != null) {
                        price = (Double) sourceAsMap.get("price");
                    }

                } catch (Exception e) {
                    e.printStackTrace();
                }
                coursePub.setPrice(price);
                // 旧价格
                Double price_old = null;
                try {
                    if (sourceAsMap.get("price_old") != null) {
                        price_old = (Double) sourceAsMap.get("price_old");
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
                coursePub.setPrice_old(price_old);
                // 将coursePub对象放入list
                list.add(coursePub);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        queryResult.setList(list);
        QueryResponseResult<CoursePub> queryResponseResult = new QueryResponseResult<CoursePub>(CommonCode.SUCCESS, queryResult);
        return queryResponseResult;
    }
}
```

