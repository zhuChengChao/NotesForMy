# Redis：学习笔记-03

> 根据 bilibili 上讲解 Redis 中观看较多的视频课程而顺带做的笔记，课程地址：[Redis最新超详细版教程通俗易懂](https://www.bilibili.com/video/BV1S54y1R7SB)，总计分为4篇，此为 3/4 篇。
>

## 7. Redis配置文件

启动的时候，就通过配置文件来启动！ 

### 7.1 Units

用来指定单位

```properties
# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same. 
# 配置文件对于大小写不敏感
```

### 7.2 Includes

添加相应的配置文件

```properties
################################## INCLUDES ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include .\path\to\local.conf
# include c:\path\to\other.conf
```

### 7.3 Network

```bash
bind 192.168.1.100 10.0.0.1  # 绑定的IP地址
protected-mode yes  		 # 保护模式
port 6379					 # 端口号
```

### 7.4 General

```bash
# 默认情况下Redis不作为守护进程运行；若需要以守护进程运行，则设置为yes
# 但是windows下不支持daemonize no
daemonize no  

# 如果以后台的方式运行，我们就需要指定一个 pid 文件; 默认为：/var/run/redis_6379.pid
pidfile /var/run/redis_6379.pid 

# Specify the server verbosity level.  指定日志等级
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice

# 指定日志文件名
logfile ""  

# 设置数据库的数量；默认数据库是db0
databases 16  
```

### 7.5 Snapshotting

持久化， 在规定的时间内，执行了多少次操作，则会持久化到文件 `.rdb / .aof`

redis 是内存数据库，如果没有持久化，那么数据断电即失

```bash
# 默认配置：save <seconds> <changes>
# 如果900s内，如果至少有一个1 key进行了修改，即进行持久化操作
save 900 1
# 如果300s内，如果至少10 key进行了修改，进行持久化操作
save 300 10
# 如果60s内，如果至少10000 key进行了修改，进行持久化操作
save 60 10000

# yes:在启用了持久化的情况下，如果持久化出错，则Redis会停止写入
stop-writes-on-bgsave-error yes  

# 是否压缩.rdb文件，yes：压缩，需要消耗一点CPU资源
rdbcompression yes  

# 保存rdb文件时，进行校验
rdbchecksum yes  

# .rdb文件名
dbfilename dump.rdb  

# rdb 文件保存的目录
dir ./  
```

### 7.6 Replication

见后续 **主从复制** 章节

### 7.7 Security

可以在这里设置 redis 的密码，默认是没有密码;

但是一般都是通过控制端设置密码：

```bash
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> config get requirepass # 获取redis的密码
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass "123456" # 设置redis的密码
OK
127.0.0.1:6379> config get requirepass # 发现所有的命令都没有权限了
(error) NOAUTH Authentication required.
127.0.0.1:6379> ping
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456 # 使用密码进行登录！
OK
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "123456"
```

### 7.8 Limits

```bash
# 设置能连接上redis的最大客户端的数量
maxclients 10000  

# redis 配置最大的内存容量
maxmemory <bytes> 

# 内存到达上限之后的处理策略
maxmemory-policy noeviction  
# volatile-lru -> 根据LRU算法删除带有expire的key
# allkeys-lru -> 根据LRU算法删除任何的key
# volatile-random -> 随机删除带有expire的key
# allkeys-random -> 随机删除
# volatile-ttl -> 删除最小ttl值的key
# noeviction -> 永不过期，直接返回错误
```

### 7.9 Append Only Mode

```bash
# 默认是不开启aof模式的，默认是使用rdb方式持久化的，在大部分所有的情况下，rdb完全够用
appendonly no 

# 持久化的文件的名字
appendfilename "appendonly.aof" 

# appendfsync always # 每次修改都会 sync，消耗性能
appendfsync everysec # 每秒执行一次 sync，但可能会丢失这1s的数据！
# appendfsync no 	 # 不执行 sync，这个时候操作系统自己同步数据，速度最快！
```

具体的配置，见 **Redis持久化** 章节

## 8. Redis 持久化

Redis 是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了持久化功能。

### 8.1 RDB

 **Redis Database：持久化快照保存流程**

![rdb保存流程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708469.PNG)

> 在主从复制中，`.rdb `一般起到备用作用，在从机上。 

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是 Snapshot 快照，它恢复时是将快照文件直接读到内存里。

Redis会单独创建（fork）一个**子进程来进行持久化**，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件**替换**上次持久化好的文件。**整个过程中，主进程是不进行任何IO操作的**。这就确保了极高的性能。

如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。**RDB的缺点是最后一次持久化后的数据可能丢失**。我们默认的就是RDB，一般情况下不需要修改这个配置。

有时候在生产环境我们会将这个文件进行备份。

**配置文件中的属性：**

rdb 默认保存的文件是 `dump.rdb` 都是在我们的配置文件中的快照选项(SNAPSHOTTING)中进行配置的

```bash
# 1.保存策略
save 900 1
save 300 10
save 60 10000
# 2.在rdb文件保存出错时，是否继续接受redis的写请求
stop-writes-on-bgsave-error yes
# 3.保存rdb文件时是否进行压缩操作
rdbcompression yes
# 4. 是否对rdb文件进行校验
rdbchecksum yes
# 5. rdb文件默认的文件名
dbfilename dump.rdb
# 6. rdb文件默认保存路径
dir ./
```

**触发机制**：

1. 在save的规则满足的情况下，会自动触发rdb规则；
2. 执行 flushall/flushdb 命令，也会触发我们的rdb规则；
3. 退出redis （shutdown），也会产生 rdb 文件

**恢复rdb文件**：

1. 只需要将rdb文件放在我们redis启动目录就可以，redis启动的时候会**自动检查** dump.rdb 恢复其中的数据； 

2. 查看需要存在的位置

   ```bash
   127.0.0.1:6379> config get dir
   1) "dir"
   2) "C:\\Software\\redis"
   ```

> 几乎就他自己默认的配置就够用了

**优点：**

1. 适合大规模的数据恢复；
2. 对数据的完整性要不高

**缺点：**

1. 需要一定的时间间隔进程操作，但如果redis意外宕机了，这个最后一次修改数据就没有的了
2. fork进程的时候，会占用一定的内存空间

### 8.2 AOF

Append Only File：**将我们的所有命令都记录下来**，类似history，**恢复的时候就把这个文件全部在执行一遍**

![aof保存流程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708028.PNG)

以日志的形式来记录每个写操作，**将Redis执行过的所有指令记录下来**（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

**配置文件中的属性：**

```bash
# 1.是否开启aof
# 默认是不开启的，修改为yes即可开启
appendonly no

# 2.aof默认的名字
appendfilename "appendonly.aof"

# 3.追加写入文件的策略
# appendfsync always  # 每次修改都写入，最慢最安全
appendfsync everysec  # 默认值，每隔1s写入
# appendfsync no	  # 让操作系统决定什么时候写入，更快

# 4. 参考：https://blog.csdn.net/jingkyks/article/details/46956905
# 在 1.重写aof文件操作 && 2.主进程写aof文件操作时，选择哪一个操作阻塞：
# no：令重写阻塞，是安全的方式；
# yes：重写aof文件不阻塞，但是主进程写aof阻塞，会出现数据丢失情况
no-appendfsync-on-rewrite no

# 5.aof大小相关属性，当aof文件超过设定值时，会触发重写aof文件操作
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

若要开启aof操作记录，令`appendonly yes`并重启redis服务器`redis-server.exe redis.windows.conf`

在redis中进行一些操作：

```bash
127.0.0.1:6379> set name zhangsan
OK
127.0.0.1:6379> set age 18
OK
127.0.0.1:6379> set sex male
OK
127.0.0.1:6379> keys *
1) "name"
2) "sex"
3) "age"
```

查看生成的持久化文件`appendonly.aof`

```aof
*2
$6
SELECT
$1
0
*3
$3
set
$4
name
$8
zhangsan
*3
$3
set
$3
age
$2
18
*3
$3
set
$3
sex
$4
male
```

删除`dump.rdb`（排除该文件的作用），重新启动redis服务器，查询之前写入的值，发现这些值还是存在的，即`appendonly.aof`发挥了作用。

**错误处理：**

在`appendonly.aof`中手动添加一些内容，使文件出错：

```
sex
$4
male 123456
```

重启服务器

```bash
C:\Software\Redis>redis-server.exe redis.windows.conf
[6084] 18 Nov 13:50:14.415 # Server started, Redis version 3.2.100
[6084] 18 Nov 13:50:14.417 # Bad file format reading the append only file: make a backup of your AOF file, then use ./redis-check-aof --fix <filename>
```

从提示中可以看出：AOF文件有格式错误，并提示可以通过`./redis-check-aof --fix <filename>`对其进行修复操作：

```bash
C:\Software\Redis>redis-check-aof.exe --fix appendonly.aof
0x              74: Expected \r\n, got: 2031
AOF analyzed: size=129, ok_up_to=90, diff=39
This will shrink the AOF from 129 bytes, with 39 bytes, to 90 bytes
Continue? [y/N]: y
Successfully truncated AOF
```

再次重启服务器，这次服务器可以成功被启动，并进行查询：

```bash
C:\Software\Redis>redis-cli.exe
127.0.0.1:6379> keys *
1) "name"
2) "age"
127.0.0.1:6379> get name
"zhangsan"
127.0.0.1:6379> get age
"18"
```

可以看到出错的内容被删除了，没有错误的内容还被保留着。

**优点：**

1. 每一次修改都同步，文件的完整会更加好；
2. 每秒同步一次，可能会丢失一秒的数据；
3. 从不同步，效率最高的；

**缺点：**

1. 相对于数据文件来说，aof 远远大于 rdb，修复的速度也比 rdb 慢；
2. aof 运行效率也要比 rdb 慢，所以我们 redis 默认的配置就是 rdb 持久化。

### 8.3 总结

1. RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储
2. AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会**重新执行这些命令来恢复原始的数据**，AOF 命令以 Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得 AOF 文件的体积不至于过大。
3. 仅仅需要缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化
4. 同时开启两种持久化方式：
   1. 在这种情况下，当redis重启的时候会**优先载入AOF文件**来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
   2. RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？作者建议不要，**因为RDB更适合用于备份数据库**（AOF在不断变化不好备份），**快速重启**，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。
5. 性能建议
   1. 因为RDB文件只用作后备用途，**建议只在Slave上持久化RDB文件**，而且只要15分钟备份一次就够了，只保留 `save 900 1` 这条规则。
   2. 如果 Enable AOF ，好处是**在最恶劣情况下也只会丢失不超过两秒数据**，启动脚本较简单，只加载自己的 AOF 文件就可以了，代价一是带来了持续的 IO，二是 AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少 AOF rewrite 的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
   3. 如果不 Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了 rewrite 时带来的系统波动。代价是如果 Master/Slave 同时宕掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个，微博就是这种架构。  

## 9. Redis 发布订阅

### 9.1 概述

Redis 发布订阅(pub/sub)是一种**消息通信模式**：发送者(pub)发送消息，订阅者(sub)接收消息。例如：微信、微博、关注系统。

Redis 客户端可以订阅任意数量的频道。

订阅/发布消息图：

第一个：消息发送者；第二个：频道；第三个：消息订阅者

![redis发布订阅](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708005.PNG)

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端：client2、client5 和 client1 之间的关系 ：

![频道与订阅者之间的关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708410.PNG)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：  

![频道与订阅者之间的关系2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708895.PNG)

### 9.2 命令

这些命令被广泛用于**构建即时通信应用**，比如网络聊天室(chatroom)和实时广播、实时提醒等。

| 序号 | 命令                                         | 描述                               |
| :--- | :------------------------------------------- | ---------------------------------- |
| 1    | PSUBSCRIBE pattern [pattern ...\]            | 订阅一个或多个符合给定模式的频道。 |
| 2    | PUBSUB subcommand [argument [argument ...\]] | 查看订阅与发布系统状态             |
| 3    | PUBLISH channel message                      | 将信息发送到指定的频道             |
| 4    | PUNSUBSCRIBE [pattern [pattern ...\]]        | 退订所有给定模式的频道             |
| 5    | SUBSCRIBE channel [channel ...\]             | 订阅给定的一个或多个频道的信息     |
| 6    | UNSUBSCRIBE [channel [channel ...\]]         | 指退订给定的频道                   |

> 参考：[菜鸟教程：Redis 发布订阅](https://www.runoob.com/redis/redis-pub-sub.html)

**测试**

1. 订阅端：

```bash
127.0.0.1:6379> subscribe jinrishuofa	# 订阅一个频道 jinrishuofa
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "jinrishuofa"
3) (integer) 1
# 等待读取推送的信息
1) "message"		# 消息
2) "jinrishuofa"	# 哪个频道的消息
3) "hello"			# 消息的具体内容
1) "message"
2) "jinrishuofa"
3) "banyemuzhujingjiaolinalian"
```

2. 发送端

```bash
127.0.0.1:6379> publish jinrishuofa "hello"
(integer) 1
127.0.0.1:6379> publish jinrishuofa "banyemuzhujingjiaolinalian"
(integer) 1
```

### 9.3 原理

Redis 是使用C实现的，通过分析 Redis 源码里的 pubsub.c 文件，了解发布和订阅机制的底层实现，籍此加深对 Redis 的理解。  

Redis 通过 PUBLISH 、SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能。

通过 SUBSCRIBE 命令订阅某频道后，redis-server 里维护了一个**字典**，字典的键就是一个个 频道，而字典的值则是一个**链表**，链表中保存了所有订阅这个 channel 的客户端，SUBSCRIBE 命令的关键，就是将客户端添加到给定 channel 的订阅链表中。  

通过 PUBLISH 命令向订阅者发送消息，redis-server 会使用给定的频道作为键，在它所维护的 channel 字典中查找记录了订阅这个频道的所有客户端的链表，**遍历这个链表**，将消息发布给所有订阅者。  

Pub/Sub 从字面上理解就是发布（Publish）与订阅（Subscribe），在Redis中，你可以设定对某一个 key 值进行消息发布及消息订阅，当一个 key 值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作**实时消息系统**，比如普通的即时聊天，群聊等功能。

**使用场景：**

1. 实时消息系统
2. 实时聊天：频道当做聊天室，将信息回显给所有人即可
3. 订阅，关注系统都是可以的

但是稍微复杂的场景我们就会使用 **消息中间件 MQ**。