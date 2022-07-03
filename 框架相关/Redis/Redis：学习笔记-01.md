# Redis：学习笔记-01

> 根据 bilibili 上讲解 Redis 中观看较多的视频课程而顺带做的笔记，课程地址：[Redis最新超详细版教程通俗易懂](https://www.bilibili.com/video/BV1S54y1R7SB)；总计分为4篇，此为 1/4 篇。
>

## 1. Redis入门

### 2.1 概述

Redis（Remote Dictionary Server )，即远程字典服务。

是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

Redis 会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了 master-slave(主从)同步。

免费和开源，是当下最热门的 NoSQL 技术之一，也被人们称之为结构化数据库。

**Redis 用途：**

1. 内存存储、持久化，内存中是断电即失，所以说持久化很重要（rdb、aof）
2. 效率高，可以用于高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计时器、计数器（浏览量）
6. ........ 

**特性：**

1. 多样的数据类型
2. 持久化
3. 集群
4. 事务
5. ...... 

### 2.2 测试性能

**redis-benchmark** 是一个压力测试工具；

官方自带的性能测试工具；

redis-benchmark 命令参数：[图片来自菜鸟教程](https://www.runoob.com/redis/redis-benchmarks.html)

![Redis性能测试](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251707311.PNG)

简单测试下：

```bash
# 命令格式：redis-benchmark [option] [option value]
C:\Software\redis>redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 10000
====== PING_INLINE ======
  10000 requests completed in 0.13 seconds
  100 parallel clients
  3 bytes payload
  keep alive: 1

31.43% <= 1 milliseconds
99.21% <= 2 milliseconds
100.00% <= 2 milliseconds
77519.38 requests per second

====== PING_BULK ======
  10000 requests completed in 0.14 seconds
  100 parallel clients
  3 bytes payload
  keep alive: 1

92.37% <= 1 milliseconds
99.79% <= 2 milliseconds
100.00% <= 2 milliseconds
74074.07 requests per second

====== SET ======
  10000 requests completed in 0.14 seconds
  100 parallel clients
  3 bytes payload
  keep alive: 1

69.40% <= 1 milliseconds
98.80% <= 2 milliseconds
100.00% <= 2 milliseconds
72463.77 requests per second
```

如何查看这些分析呢？ 

```bash
====== GET ======
  10000 requests completed in 0.18 seconds  # 完成1w个请求的处理耗时
  100 parallel clients						# 100个并发客户端
  3 bytes payload							# 每次写入/读3个字节
  keep alive: 1								# 只有一台服务器来处理这些请求，单机性能

82.10% <= 1 milliseconds					# 1ms内完成的读情况情况
96.16% <= 2 milliseconds
98.66% <= 3 milliseconds
99.20% <= 4 milliseconds
99.21% <= 10 milliseconds
99.37% <= 11 milliseconds
99.65% <= 12 milliseconds
99.78% <= 13 milliseconds
99.91% <= 14 milliseconds
99.97% <= 15 milliseconds
100.00% <= 15 milliseconds					# 在15ms时完成请求的处理
54945.05 requests per second				# 每秒处理54945.05次请求
```

### 2.3 基础的知识 

Redis默认有16个数据库：

> redis.windows.conf配置文件中：

```bash
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
```

默认使用的是第0个

可以使用 select 进行切换数据库

```bash
127.0.0.1:6379> select 3  # 切换数据库
OK
127.0.0.1:6379[3]> dbsize # 查看DB大小
(integer) 0
```

清除当前数据库 `flushdb`

```bash
127.0.0.1:6379[3]> dbsize
(integer) 0
127.0.0.1:6379[3]> set name zhangsan
OK
127.0.0.1:6379[3]> dbsize
(integer) 1
127.0.0.1:6379[3]> flushdb
OK
127.0.0.1:6379[3]> dbsize
(integer) 0
```

清除全部数据库的内容 `flushall`

**Redis 是单线程的**

明白Redis是很快的，官方表示，Redis是基于内存操作，CPU不是Redis性能瓶颈，Redis的瓶颈是根据机器的内存和网络带宽，既然可以使用单线程来实现，就使用单线程了。

Redis 是 C 语言写的，官方提供的数据为 100000+ 的QPS(Query Per Second, 每秒查询率)，完全不比同样是使用 key-vale的Memecache差。

**Redis 为什么单线程还这么快？**

1. 误区1：高性能的服务器一定是多线程的？
2. 误区2：多线程（CPU上下文会切换）一定比单线程效率高。

**核心**：redis 是将所有的数据全部放在内存中的，所以说使用单线程去操作效率就是最高的，多线程（CPU上下文会切换：耗时的操作），对于内存系统来说，如果没有上下文切换效率就是最高的。多次读写都是在一个CPU上的，在内存充足的情况下，这个就是最佳的方案。

## 2. 五大数据类型 

官网文档：

![redis官网介绍](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251707490.PNG)

中文官网：

![redis官网介绍cn](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251707159.PNG)

### 2.1 Redis:Key操作

涉及命令：**keys/set/get/exists/move/ttl/type**

| 命令   | 说明                  | 示例                |
| ------ | --------------------- | ------------------- |
| keys   | 查看对应的key         | keys *              |
| set    | 设置值                | set key value       |
| get    | 获取值                | get key             |
| exists | 判断当前的key是否存在 | exists key          |
| move   | 移动key到另一个数据库 | move key 数据库序号 |
| ttl    | 查看当前key的剩余时间 | ttl key             |
| type   | 查看当前key的一个类型 | type key            |

使用示例：

```bash
127.0.0.1:6379> keys *  			# 查看所有的key
(empty list or set)
127.0.0.1:6379> set name zhangsan	# set key
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> set age 18
OK
127.0.0.1:6379> keys *
1) "age"
2) "name"
127.0.0.1:6379> exists name			# 判断当前的key是否存在
(integer) 1
127.0.0.1:6379> exists names
(integer) 0
127.0.0.1:6379> move name 1			# 移动key到另一个数据库
127.0.0.1:6379> keys *
1) "age"
127.0.0.1:6379> select 1            # select 数据库号，选择第几个数据库
OK
127.0.0.1:6379[1]> keys *
1) "name"
127.0.0.1:6379[1]> get name
"zhangsan"
127.0.0.1:6379[1]> expire name 10	# 设置key的过期时间，单位是秒
(integer) 1
127.0.0.1:6379[1]> ttl name			# 查看当前key的剩余时间
(integer) 7
127.0.0.1:6379[1]> ttl name
(integer) 3
127.0.0.1:6379[1]> ttl name
(integer) -2
127.0.0.1:6379[1]> get name
(nil)
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> type age			# 查看当前key的一个类型
string
```

后面如果遇到不会的命令，可以在官网查看[帮助文档](http://www.redis.cn/documentation.html)

### 2.2 String

1. 涉及命令：**set/get/exists/append/strlen**

   | 命令   | 说明                                           | 示例             |
   | ------ | ---------------------------------------------- | ---------------- |
   | set    | 设置值                                         | set key value    |
   | get    | 获得值                                         | get key          |
   | exists | 判断当前的key是否存在                          | exists key       |
   | append | 追加字符串，如果当前key不存在，就相当于set key | append key value |
   | strlen | 获取字符串的长度                               | strlen key       |

   使用示例：

   ```bash
   127.0.0.1:6379> set key1 v1			# 设置值
   OK
   127.0.0.1:6379> get key1			# 获得值
   "v1"
   127.0.0.1:6379> keys *				# 获得所有的key
   1) "key1"
   127.0.0.1:6379> append key1 hello	# 追加字符串，如果当前key不存在，就相当于set key
   (integer) 7
   127.0.0.1:6379> get key1		
   "v1hello"
   127.0.0.1:6379> strlen key1			# 获取字符串的长度
   (integer) 7
   127.0.0.1:6379> append key1 ,world
   (integer) 13
   127.0.0.1:6379> strlen key1
   (integer) 13
   127.0.0.1:6379> get key1
   "v1hello,world"
   127.0.0.1:6379> exists key1			# 判断某一个key是否存在
   (integer) 1
   ```

2. 涉及命令：**incr/decr/incrby/decrby**

   | 命令   | 说明                       | 示例            |
   | ------ | -------------------------- | --------------- |
   | incr   | 变量自增1                  | incr views      |
   | decr   | 变量自减1                  | decr views      |
   | incrby | 指定增量，即可以设置步长   | incrby views 10 |
   | decrby | 指定减少量，即可以设置步长 | decrby views 5  |

   使用示例

   ```bash
   127.0.0.1:6379> set views 0			# 初始浏览量为0
   OK
   127.0.0.1:6379> get views
   "0"
   127.0.0.1:6379> incr views			# 自增1 浏览量变为1
   (integer) 1
   127.0.0.1:6379> get views
   "1"
   127.0.0.1:6379> decr views			# 自减1 浏览量0
   (integer) 0
   127.0.0.1:6379> incrby views 10		# 可以设置步长，指定增量
   (integer) 10
   127.0.0.1:6379> get views
   "10"
   127.0.0.1:6379> decrby views 5		# 可以设置步长,指定减少量
   (integer) 5
   127.0.0.1:6379> get views
   "5"
   ```

3. 涉及命令：**getrange/setrange**

   | 命令     | 说明                     | 示例               |
   | -------- | ------------------------ | ------------------ |
   | getrange | 获取一定长度的字符串     | getrange key1 0 3  |
   | setrange | 替换指定位置开始的字符串 | setrange key2 1 xx |

   使用示例：

    ```bash
    127.0.0.1:6379> set key1 "hello, world"		# 设置 key1 的值
    OK
    127.0.0.1:6379> get key1
    "hello, world"
    127.0.0.1:6379> getrange key1 0 3			# 截取字符串 [0,3]
    "hell"
    127.0.0.1:6379> getrange key1 0 -1			# 获取全部的字符串 和 get key是一样的
    "hello, world"
    127.0.0.1:6379> set key2 abcdefg
    OK
    127.0.0.1:6379> get key2
    "abcdefg"
    127.0.0.1:6379> setrange key2 1 xx			# 替换指定位置开始的字符串
    (integer) 7
    127.0.0.1:6379> get key2
    "axxdefg"
    ```

4. 涉及命令：**setex/setnx**

   | 命令  | 说明                           | 示例                        |
   | ----- | ------------------------------ | --------------------------- |
   | setex | 设置过期时间，set with expire  | setex key1 10 "hello,world" |
   | setnx | 不存在在设置，set if not exist | setnx key2 "hello,redis"    |

   使用示例：

    ```bash
    # setex (set with expire) # 设置过期时间
    # setnx (set if not exist) # 不存在在设置 （在分布式锁中会常常使用）
    127.0.0.1:6379> setex key1 10 "hello,world"		# 设置key1的值,10秒后过期
    OK
    127.0.0.1:6379> ttl key1
    (integer) 5
    127.0.0.1:6379> ttl key1
    (integer) -2
    127.0.0.1:6379> setnx key2 "hello,redis"		# 如果key2不存在，创建key2
    (integer) 1
    127.0.0.1:6379> keys *
    1) "key2"
    127.0.0.1:6379> setnx key2 "fuck,redis"			# 如果key2存在，创建失败
    (integer) 0
    127.0.0.1:6379> get key2
    "hello,redis"
    ```

5. 涉及命令:**mset/mget/msetnx**

   | 命令   | 说明                                                         | 示例                   |
   | ------ | ------------------------------------------------------------ | ---------------------- |
   | mset   | 同时设置多个值                                               | mset k1 v1 k2 v2 k3 v3 |
   | mget   | 同时获取多个值                                               | mget k1 k2 k3          |
   | msetnx | 不存在时批量设置<br />是一个原子性的操作，要么一起成功，要么一起失败 | msetnx k1 v1 k4 v4     |

   使用示例：

    ```bash
    127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3	# 同时设置多个值
    OK
    127.0.0.1:6379> keys *
    1) "k2"
    2) "k1"
    3) "k3"
    127.0.0.1:6379> mget k1 k2 k3			# 同时获取多个值
    1) "v1"
    2) "v2"
    3) "v3"
    127.0.0.1:6379> msetnx k1 v1 k4 v4		# msetnx 是一个原子性的操作，要么一起成功，要么一起失败
    (integer) 0
    127.0.0.1:6379> get k4
    (nil)
   
    # 对象保存，通过json格式
    127.0.0.1:6379> set user:1 {name:zhangsan,age:3}
    OK
    127.0.0.1:6379> get user:1
    "{name:zhangsan,age:3}"
    # 这里的key是一个巧妙的设计： user:{id}:{filed} , 如此设计在Redis中是完全OK
    127.0.0.1:6379> mset user:1:name zhangsan user:1:age 2  
    OK
    127.0.0.1:6379> mget user:1:name user:1:age
    1) "zhangsan"
    2) "2"
    127.0.0.1:6379> keys *
    1) "user:1"
    2) "user:1:name"
    3) "user:1:age"
    ```

6. 涉及命令:**getset**

   | 命令   | 说明                                   | 示例               |
   | ------ | -------------------------------------- | ------------------ |
   | getset | 如果存在值，获取原来的值，并设置新的值 | getset key "nihao" |

   使用示例
   
    ```bash
    127.0.0.1:6379> set key "hello"			# 如果存在值，获取原来的值，并设置新的值
    OK
    127.0.0.1:6379> getset key "nihao"
    "hello"
    127.0.0.1:6379> getset key1 "world"		# 如果不存在值，则返回 nil
    (nil)
    127.0.0.1:6379> get key1
    "world"                                                                   
    ```

String类似的使用场景：value除了是我们的字符串还可以是我们的数字

* 计数器
* 统计多单位的数量
* 对象缓存存储

### 2.3 List

**基本的数据类型，列表** 

在 Redis 里面，我们可以把list玩成 ：栈、队列、阻塞队列

所有的 list 命令基本都是用L开头的，Redis不区分大小命令

1. 涉及命令：**lpush/lrange/rpush**

   | 命令   | 说明                                      | 示例             |
   | ------ | ----------------------------------------- | ---------------- |
   | lpush  | 将一个值或者多个值，插入到列表头部 （左） | lpush list one   |
   | lrange | 通过区间获取具体的值                      | lrange list 0 1  |
   | rpush  | 将一个值或者多个值，插入到列表尾部 （右） | rpush list right |

   使用示例：

    ```bash
    127.0.0.1:6379> lpush list one		# 将一个值或者多个值，插入到列表头部 （左）
    (integer) 1
    127.0.0.1:6379> lpush list two
    (integer) 2
    127.0.0.1:6379> lpush list three
    (integer) 3
    127.0.0.1:6379> lrange list 0 -1	# 获取list中值
    1) "three"
    2) "two"
    3) "one"
    127.0.0.1:6379> lrange list 0 1		# 通过区间获取具体的值
    1) "three"
    2) "two"
    127.0.0.1:6379> rpush list right	# 将一个值或者多个值，插入到列表尾部 （右）
    (integer) 4
    127.0.0.1:6379> lrange list 0 -1
    1) "three"
    2) "two"
    3) "one"
    4) "right"
    ```

2. 涉及命令：**lpop/rpop**

    | 命令 | 说明                         | 示例      |
    | ---- | ---------------------------- | --------- |
    | lpop | 移除list的第一个元素（左）   | lpop list |
    | rpop | 移除list的最后一个元素（右） | rpop list |

    使用示例：

    ```bash
    127.0.0.1:6379> lrange list 0 -1
    1) "three"
    2) "two"
    3) "one"
    4) "right"
    127.0.0.1:6379> lpop list		# 移除list的第一个元素
    "three"
    127.0.0.1:6379> rpop list		# 移除list的最后一个元素
    "right"
    127.0.0.1:6379> lrange list 0 -1
    1) "two"
    2) "one"
    ```

3. 涉及命令：**lindex/llen**

    | 命令   | 说明                           | 示例          |
    | ------ | ------------------------------ | ------------- |
    | lindex | 通过下标获得 list 中的某一个值 | lindex list 0 |
    | llen   | 返回列表的长度                 | llen list     |

    使用示例：

    ```bash
    127.0.0.1:6379> lrange list 0 -1
    1) "two"
    2) "one"
    127.0.0.1:6379> lindex list 0		# 通过下标获得 list 中的某一个值
    "two"
    127.0.0.1:6379> lindex list 1
    "one"
    127.0.0.1:6379> llen list			# 返回列表的长度
    (integer) 2
    ```

4. 涉及命令：**lrem**

    | 命令 | 说明                                    | 示例            |
    | ---- | --------------------------------------- | --------------- |
    | lrem | 移除list集合中指定个数的value，精确匹配 | lrem list 1 one |

    使用示例：

    ```bash
    127.0.0.1:6379> lpush list one one two two three four
    (integer) 6
    127.0.0.1:6379> lrange list 0 -1
    1) "four"
    2) "three"
    3) "two"
    4) "two"
    5) "one"
    6) "one"
    127.0.0.1:6379> lrem list 1 one		# 移除list集合中指定个数的value，精确匹配
    (integer) 1
    127.0.0.1:6379> lrange list 0 -1
    1) "four"
    2) "three"
    3) "two"
    4) "two"
    5) "one"
    127.0.0.1:6379> lrem list 2 two
    (integer) 2
    127.0.0.1:6379> lrange list 0 -1
    1) "four"
    2) "three"
    3) "one"
    ```

5. 涉及命令：**trim**

    | 命令 | 说明                   | 示例           |
    | ---- | ---------------------- | -------------- |
    | trim | 通过下标截取指定的长度 | ltrim list 0 4 |

    使用示例：

    ```bash
    127.0.0.1:6379> rpush list 1 2 3 4 5 6 7 8
    (integer) 8
    127.0.0.1:6379> lrange list 0 -1
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    6) "6"
    7) "7"
    8) "8"
    127.0.0.1:6379> ltrim list 0 4	# 通过下标截取指定的长度，这个list已经被改变了，截断了只剩下截取的元素
    OK
    127.0.0.1:6379> lrange list 0 -1
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    ```

6. 涉及命令：**rpoplpush** 

    | 命令      | 说明                                               | 示例                  |
    | --------- | -------------------------------------------------- | --------------------- |
    | rpoplpush | 移除左边列表的最后一个元素，将他移动到右边的列表中 | rpoplpush list1 list2 |

    使用示例：

    ```bash
    127.0.0.1:6379> rpush list1 1 2 3
    (integer) 3
    127.0.0.1:6379> rpush list2 3 4 5
    (integer) 3
    127.0.0.1:6379> rpoplpush list1 list2	# 移除列表的最后一个元素，将他移动到新的列表中
    "3"
    127.0.0.1:6379> lrange list1 0 -1
    1) "1"
    2) "2"
    127.0.0.1:6379> lrange list2 0 -1
    1) "3"
    2) "3"
    3) "4"
    4) "5"
    ```

7. 涉及命令：**lset**

    | 命令 | 说明                                                         | 示例           |
    | ---- | ------------------------------------------------------------ | -------------- |
    | lset | 将列表中指定下标的值替换为另外一个值，更新操作<br />如果存在，更新当前下标的值<br />如果不存在，则会报错 | lset list 0 10 |

    使用示例：

    ```bash
    # 将列表中指定下标的值替换为另外一个值，更新操作
    127.0.0.1:6379> lrange list 0 -1
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    127.0.0.1:6379> lset list 0 10		# 如果存在，更新当前下标的值
    OK
    127.0.0.1:6379> lset list 5 60		# 如果不存在，则会报错
    (error) ERR index out of range
    127.0.0.1:6379> lrange list 0 -1
    1) "10"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    ```

8. 涉及命令：**linsert **

    | 命令    | 说明                                                  | 示例                                                     |
    | ------- | ----------------------------------------------------- | -------------------------------------------------------- |
    | linsert | 将某个具体的value插入到列把你中某个元素的前面或者后面 | linsert list before 10 -10<br />linsert list after 10 20 |

    使用示例：

    ```bash
    # 将某个具体的value插入到列把你中某个元素的前面或者后面
    127.0.0.1:6379> lrange list 0 -1
    1) "10"
    127.0.0.1:6379> linsert list before 10 -10
    (integer) 2
    127.0.0.1:6379> linsert list after 10 20
    (integer) 3
    127.0.0.1:6379> lrange list 0 -1
    1) "-10"
    2) "10"
    3) "20"
    ```

**小结：**

* 实际上是一个链表，before，after，left，right 都可以插入值
* 如果key不存在，创建新的链表
* 如果key存在，新增内容 
* 如果移除了所有值，空链表，也代表不存在
* 在两边插入或者改动值，效率最高， 中间元素，相对来说效率会低一点

消息排队、消息队列 （Lpush Rpop）、栈（ Lpush Lpop）

### 2.4 Set 

set中的值是不能重复的 

1. 涉及命令：**sadd/smembers/sismember/scard/srem**

   | 命令      | 说明                          | 示例                |
   | --------- | ----------------------------- | ------------------- |
   | sadd      | 向set中添加元素               | sadd set "hello"    |
   | smembers  | 查看指定set的所有值           | smembers set        |
   | sismember | 判断某一个值是不是在set集合中 | sismember set hello |
   | scard     | 获取set集合中的内容元素个数   | scard set           |
   | srem      | 移除set集合中的指定元素       | srem set hello      |

   使用示例：

    ```bash
    127.0.0.1:6379> sadd set "hello"
    (integer) 1
    127.0.0.1:6379> sadd set "world"
    (integer) 1
    127.0.0.1:6379> sadd set "hello" "redis"
    (integer) 1
    127.0.0.1:6379> smembers set		# 查看指定set的所有值
    1) "redis"
    2) "world"
    3) "hello"
    127.0.0.1:6379> sismember set hello	# 判断某一个值是不是在set集合中
    (integer) 1
    127.0.0.1:6379> sismember set dog
    (integer) 0
    127.0.0.1:6379> scard set			# 获取set集合中的内容元素个数
    (integer) 3
    127.0.0.1:6379> srem set hello		# 移除set集合中的指定元素
    (integer) 1
    127.0.0.1:6379> scard set
    (integer) 2
    127.0.0.1:6379> smembers set
    1) "redis"
    2) "world"
    ```

2. 涉及命令：**srandmember/spop**

    | 命令        | 说明                        | 示例                         |
    | ----------- | --------------------------- | ---------------------------- |
    | srandmember | 随机抽选出指定元素元素      | srandmember set 指定元素个数 |
    | spop        | 随机删除一些set集合中的元素 | spop set<br />spop set 2     |

    使用示例：

    ```bash
    127.0.0.1:6379> sadd set 1 2 3 4 5 6
    (integer) 6
    127.0.0.1:6379> srandmember set 1	# 随机抽选出一个元素
    1) "2"
    127.0.0.1:6379> srandmember set 1
    1) "1"
    127.0.0.1:6379> srandmember set 3	# 随机抽选出指定个数的元素
    1) "4"
    2) "3"
    3) "2"
    127.0.0.1:6379> srandmember set 3
    1) "6"
    2) "1"
    3) "2"
    127.0.0.1:6379> spop set			# 随机删除一些set集合中的元素
    "4"
    127.0.0.1:6379> spop set 2
    1) "2"
    2) "1"
    127.0.0.1:6379> smembers set
    1) "3"
    2) "5"
    3) "6"
    ```

3. 涉及命令：**smove**

    | 命令  | 说明                                  | 示例              |
    | ----- | ------------------------------------- | ----------------- |
    | smove | 将一个指定的值，移动到另外一个set集合 | smove set1 set2 3 |

    使用示例：
    ```bash
    127.0.0.1:6379> sadd set1 1 2 3
    (integer) 3
    127.0.0.1:6379> sadd set2 4 5 6
    (integer) 3
    127.0.0.1:6379> smove set1 set2 3  # 将一个指定的值，移动到另外一个set集合
    (integer) 1
    127.0.0.1:6379> smembers set1
    1) "1"
    2) "2"
    127.0.0.1:6379> smembers set2
    1) "3"
    2) "4"
    3) "5"
    4) "6"
    ```

4. 涉及命令：**sdiff/sinter/sunion**

    | 命令   | 说明   | 示例             |
    | ------ | ------ | ---------------- |
    | sdiff  | 求差集 | sdiff set1 set2  |
    | sinter | 求交集 | sinter set1 set2 |
    | sunion | 求并集 | sunion set1 set2 |

    使用示例：
    ```bash
    127.0.0.1:6379> sadd set1 1 2 3 4 5
    (integer) 5
    127.0.0.1:6379> sadd set2 4 5 6 7 8
    (integer) 5
    127.0.0.1:6379> sdiff set1 set2		# 差集
    1) "1"
    2) "2"
    3) "3"
    127.0.0.1:6379> sdiff set2 set1
    1) "6"
    2) "7"
    3) "8"
    127.0.0.1:6379> sinter set1 set2	# 交集
    1) "4"
    2) "5"
    127.0.0.1:6379> sunion set1 set2	# 并集
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    6) "6"
    7) "7"
    8) "8"
    ```

微博，A用户将所有关注的人放在一个set集合中，将它的粉丝也放在一个集合中。

共同关注，共同爱好，二度好友，推荐好友

### 2.5 Hash

**Map集合**，key-map 适合这个值是一个map集合，本质和String类型没有太大区别，还是一个简单的 key-vlaue 

1. 涉及命令：**hset/hget/hmset/hmget/hgetall/hdel/hlen/hexists/hkeys/hvals**

    使用示例：

    | 命令    | 说明                                        | 示例                                   |
    | ------- | ------------------------------------------- | -------------------------------------- |
    | hset    | set一个具体 key-vlaue                       | hset hash field1 value1                |
    | hget    | 获取一个字段值                              | hget hash field1                       |
    | hmset   | set多个 key-vlaue                           | hmset hash field2 value2 field3 value3 |
    | hmget   | 获取多个字段值                              | hmget hash field1 field2 field3        |
    | hgetall | 获取全部的数据                              | hgetall hash                           |
    | hdel    | 删除hash指定key字段,对应的value值也就消失了 | hdel hash field3                       |
    | hlen    | 获取hash表的字段数量                        | hlen hash                              |
    | hexists | 判断hash中指定字段是否存在                  | hexists hash field1                    |
    | hkeys   | 只获得所有field                             | hkeys hash                             |
    | hvals   | 只获得所有value                             | hvals hash                             |

    ```bash
    127.0.0.1:6379> hset hash field1 value1		# set一个具体 key-vlaue
    (integer) 1
    127.0.0.1:6379> hget hash field1			# 获取一个字段值
    "value1"
    127.0.0.1:6379> hmset hash field2 value2 field3 value3	# set多个 key-vlaue
    OK
    127.0.0.1:6379> hmget hash field1 field2 field3			# 获取多个字段值
    1) "value1"
    2) "value2"
    3) "value3"
    127.0.0.1:6379> hgetall hash				# 获取全部的数据
    1) "field1"
    2) "value1"
    3) "field2"
    4) "value2"
    5) "field3"
    6) "value3"
    127.0.0.1:6379> hdel hash field3			# 删除hash指定key字段,对应的value值也就消失了
    (integer) 1
    127.0.0.1:6379> hgetall hash
    1) "field1"
    2) "value1"
    3) "field2"
    4) "value2"
    127.0.0.1:6379> hlen hash					# 获取hash表的字段数量
    (integer) 2
    127.0.0.1:6379> hexists hash field1			# 判断hash中指定字段是否存在
    (integer) 1
    127.0.0.1:6379> hexists hash field3
    (integer) 0
    127.0.0.1:6379> hkeys hash					# 只获得所有field
    1) "field1"
    2) "field2"
    127.0.0.1:6379> hvals hash					# 只获得所有value
    1) "value1"
    2) "value2"
    ```

2. 涉及命令：**hincrby/hsetnx**

    | 命令    | 说明                                         | 示例                      |
    | ------- | -------------------------------------------- | ------------------------- |
    | hincrby | 指定增量                                     | hincrby hash field3 3     |
    | hsetnx  | 如果不存在则可以设置<br />如果存在则不能设置 | hsetnx hash field4 value4 |
    
    使用示例：
    ```bash
    127.0.0.1:6379> hset hash field3 3
    (integer) 1
    127.0.0.1:6379> hincrby hash field3 3		# 指定增量
    (integer) 6
    127.0.0.1:6379> hget hash field3
    "6"
    127.0.0.1:6379> hincrby hash field3 -6
    (integer) 0
    127.0.0.1:6379> hget hash field3
    "0"
    127.0.0.1:6379> hsetnx hash field4 value4	# 如果不存在则可以设置
    (integer) 1
    127.0.0.1:6379> hsetnx hash field4 value4-4	# 如果存在则不能设置
    (integer) 0
    127.0.0.1:6379> hget hash field4
    "value4"
    ```

hash变更的数据 user name age，尤其是是用户信息之类的，经常变动的信息，hash 更适合于对象的存储，String更加适合字符串存储 

### 2.6 Zset

在set的基础上，增加了一个source值

1. 涉及命令：**zadd/zrange/zrem/zcard**

    | 命令   | 说明                     | 示例                                         |
    | ------ | ------------------------ | -------------------------------------------- |
    | zadd   | 添加一个值/多个值        | zadd zset 1 one<br />zadd zset 2 two 3 three |
    | zrange | 获取指定范围内的值       | zrange zset 0 -1                             |
    | zrem   | 移除有序集合中的指定元素 | zrem zset one                                |
    | zcard  | 获取有序集合中的个数     | zcard zset                                   |

    使用示例：
    ```bash
    127.0.0.1:6379> zadd zset 1 one			# 添加一个值
    (integer) 1
    127.0.0.1:6379> zadd zset 2 two 3 three	# 添加多个值
    (integer) 2
    127.0.0.1:6379> zrange zset 0 -1
    1) "one"
    2) "two"
    3) "three"
    127.0.0.1:6379> zrem zset one			# 移除有序集合中的指定元素
    (integer) 1
    127.0.0.1:6379> zrange zset 0 -1
    1) "two"
    2) "three"
    127.0.0.1:6379> zcard zset				# 获取有序集合中的个数
    (integer) 2
    ```

2. 涉及命令：**zrangebyscore/zrevrange**

    | 命令          | 说明           | 示例                         |
    | ------------- | -------------- | ---------------------------- |
    | zrangebyscore | 显示全部的用户 | zrangebyscore zset -inf +inf |
    | zrevrange     | 从大到进行排序 | zrevrange zset 0 -1          |

    使用示例：

    ```bash
    127.0.0.1:6379> zrangebyscore zset -inf +inf	# 显示全部的用户 从小到大
    1) "one"
    2) "two"
    3) "three"
    127.0.0.1:6379> zrevrange zset 0 -1				# 从大到进行排序
    1) "three"
    2) "two"
    3) "one"
    127.0.0.1:6379> zrangebyscore zset -inf +inf withscores	# 显示全部的用户并且附带成绩
    1) "one"
    2) "1"
    3) "two"
    4) "2"
    5) "three"
    6) "3"
    127.0.0.1:6379> zrangebyscore zset -inf 2 withscores	# 显示score小于2的成员带score显示
    1) "one"
    2) "1"
    3) "two"
    4) "2"
    ```

3. 涉及命令：**zcount**

    | 命令   | 说明                   | 示例            |
    | ------ | ---------------------- | --------------- |
    | zcount | 获取指定区间的成员数量 | zcount zset 0 1 |

    使用示例：
    ```bash
    127.0.0.1:6379> zrange zset 0 -1
    1) "two"
    2) "three"
    127.0.0.1:6379> zcount zset 0 1		# 获取指定区间的成员数量
    (integer) 0
    127.0.0.1:6379> zcount zset 2 3
    (integer) 2
    127.0.0.1:6379> zcount zset 2 2
    (integer) 1
    ```

## 3. 三种特殊数据类型 

### 3.1 Geospatial

Geospatial：地理位置

朋友的定位，附近的人，打车距离计算...

Redis 的 Geo 在 Redis3.2 版本就推出了，这个功能可以推算地理位置的信息，两地之间的距离，方圆几里的人。

只有六个命令： 

![geospatial命令](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251707863.PNG)

具体查看：[官方文档](http://www.redis.cn/commands.html#geo)

1. **geoadd**：`GEOADD key longitude latitude member [longitude latitude member ...]`

    ```bash
    # 将指定的地理空间位置（经度、纬度、名称）添加到指定的key中
    # 该命令以采用标准格式的参数x,y,所以经度必须在纬度之前
    # 	有效的经度从-180度到180度
    # 	有效的纬度从-85.05112878度到85.05112878度
    # 当坐标位置超出上述指定范围时，该命令将会返回一个错误
    127.0.0.1:6379> geoadd china:city 116.41 39.91 beijing
    (integer) 1
    127.0.0.1:6379> geoadd china:city 121.43 34.50 shanghai 113.23 23.16 guangzhou
    (integer) 2
    127.0.0.1:6379> geoadd china:city 120.20 30.26 hangzhou
    (integer) 1
    127.0.0.1:6379> geoadd china:city 87.68 43.76 wulumuqi
    (integer) 1
    ```

2. **geopos**：`GEOPOS key member [member ...]`

    ```bash
    127.0.0.1:6379> geopos china:city shanghai
    1) 1) "121.42999917268753"
       2) "34.499999717161309"
    127.0.0.1:6379> geopos china:city beijing hangzhou guangzhou
    1) 1) "116.4099982380867"
       2) "39.909999566644508"
    2) 1) "120.20000249147415"
       2) "30.259999272896209"
    3) 1) "113.22999805212021"
       2) "23.159999437635349"
    ```

3. **geodist**：`GEODIST key member1 member2 [unit]`

    返回两个给定位置之间的距离。

    如果两个位置之间的其中一个不存在， 那么命令返回空值。

    指定单位的参数 unit 必须是以下单位的其中一个：

    * **m** 表示单位为米。
    * **km** 表示单位为千米。
    * **mi** 表示单位为英里。
    * **ft** 表示单位为英尺。

    如果用户没有显式地指定单位参数， 那么 `GEODIST` 默认使用米作为单位。

    ```bash
    127.0.0.1:6379> geodist china:city shanghai hangzhou km
    "485.5319"
    127.0.0.1:6379> geodist china:city shanghai wulumuqi km
    "3063.6209"
    127.0.0.1:6379> geodist china:city shanghai beijing km
    "747.9376"
    ```

4. **georadius** ：`GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]`

    以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。

    范围可以使用的单位同上；

    在给定以下可选项时， 命令会返回额外的信息：

    - `WITHDIST`: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。 距离的单位和用户给定的范围单位保持一致。
    - `WITHCOORD`: 将位置元素的经度和维度也一并返回。
    - `WITHHASH`: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。

    命令默认返回未排序的位置元素。 通过以下两个参数， 用户可以指定被返回位置元素的排序方式：

    - `ASC`: 根据中心的位置， 按照从近到远的方式返回位置元素。
    - `DESC`: 根据中心的位置， 按照从远到近的方式返回位置元素。

    ```bash
    127.0.0.1:6379> georadius china:city 120 30 100 km  # 以110，30 这个经纬度为中心 找方圆100km内的城市
    1) "hangzhou"
    127.0.0.1:6379> georadius china:city 120 30 1000 km	# 以110，30 这个经纬度为中心 找方圆1000km内的城市
    1) "hangzhou"
    2) "shanghai"
    127.0.0.1:6379> georadius china:city 120 30 10000 km # 以110，30 这个经纬度为中心 找方圆10000km内的城市
    1) "wulumuqi"
    2) "guangzhou"
    3) "hangzhou"
    4) "shanghai"
    5) "beijing"
    127.0.0.1:6379> georadius china:city 120 30 1000 km withdist  # 同时返回距离
    1) 1) "hangzhou"
       2) "34.7342"
    2) 1) "shanghai"
       2) "518.2591"
    127.0.0.1:6379> georadius china:city 120 30 1000 km withdist withcoord  # 同时返回距离+坐标
    1) 1) "hangzhou"
       2) "34.7342"
       3) 1) "120.20000249147415"
          2) "30.259999272896209"
    2) 1) "shanghai"
       2) "518.2591"
       3) 1) "121.42999917268753"
          2) "34.499999717161309"
    127.0.0.1:6379> georadius china:city 120 30 1000 km count 1  # 只返回一个
    1) "hangzhou"
    ```

5. **georadiusbymember**：`key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]`

    这个命令和 GEORADIUS命令一样， 都可以找出位于指定范围内的元素， 但是 `GEORADIUSBYMEMBER` 的中心点是由给定的位置元素决定的， 而不是像 GEORADIUS 那样， 使用输入的经度和纬度来决定中心点指定成员的位置被用作查询的中心。

    ```bash
    127.0.0.1:6379> georadiusbymember china:city hangzhou 1000 km
    1) "hangzhou"
    2) "shanghai"
    127.0.0.1:6379> georadiusbymember china:city hangzhou 1000 km withcoord
    1) 1) "hangzhou"
       2) 1) "120.20000249147415"
          2) "30.259999272896209"
    2) 1) "shanghai"
       2) 1) "121.42999917268753"
          2) "34.499999717161309"
    ```

6. **geohash**：`GEOHASH key member [member ...]`

    该命令将返回11个字符的Geohash字符串

    ```bash
    127.0.0.1:6379> geohash china:city hangzhou
    1) "wtmkphwvr00"
    127.0.0.1:6379> geohash china:city shanghai
    1) "wwnk70w9h40"
    127.0.0.1:6379> geohash china:city beijing
    1) "wx4g0crhte0"
    127.0.0.1:6379> geohash china:city wulumuqi
    1) "tzy2gnq8kp0"
    ```

**其他：**

GEO 底层的实现原理其实就是 Zset！我们可以使用Zset命令来操作geo：

```bash
127.0.0.1:6379> zrange china:city 0 -1		# 查看地图中全部的元素
1) "wulumuqi"
2) "guangzhou"
3) "hangzhou"
4) "shanghai"
5) "beijing"
127.0.0.1:6379> zrem china:city guangzhou	# 移除指定元素
(integer) 1
127.0.0.1:6379> zrange china:city 0 -1 withscores
1) "wulumuqi"
2) "3846741228821508"
3) "hangzhou"
4) "4054134192466832"
5) "shanghai"
6) "4066919237671184"
7) "beijing"
8) "4069885553085665"
```

### 3.2 Hyperloglog

**基数：**

A = {1, 3, 5, 7, 8, 7}

B = {1, 3, 4, 5, 6, 8}

不重复的元素个数：2

**简介** ：

Redis 2.8.9 版本就更新了 Hyperloglog 数据结构；

Redis Hyperloglog 基数统计的算法；

优点：占用的内存是固定，HyperLogLog 可以使用固定且很少的内存（每个 HyperLogLog 结构需要12K字节再加上key本身的几个字节）来存储集合的唯一元素.

> 网页的 UV （一个人访问一个网站多次，但是还是算作一个人）
>
> * 传统的方式， set 保存用户的 id，然后就可以统计 set 中的元素数量作为标准判断；
> * 这个方式如果保存大量的用户 id，就会比较麻烦，我们的目的是为了计数，而不是保存用户 id；
> * Hyperloglog 存在 0.81% 错误率，统计 UV 任务，可以忽略不计的； 

**命令使用：**

1. **pfadd**：`key element [element ...]`

    将除了第一个参数以外的参数存储到以第一个参数为变量名的HyperLogLog结构中.

    ```bash
    127.0.0.1:6379> pfadd mykey a b c d e f g h i j		# 创建第一组元素 mykey
    (integer) 1
    127.0.0.1:6379> pfadd yourkey i j u v w x y z		# 创建第一组元素 yourkey
    (integer) 1
    ```

2. **pfcount**：`PFCOUNT key [key ...]`

    当参数为一个key时，返回存储在HyperLogLog结构体的该变量的近似基数，如果该变量不存在，则返回0.

    当参数为多个key时，返回这些HyperLogLog并集的近似基数，这个值是将所给定的所有key的HyperLoglog结构合并到一个临时的HyperLogLog结构中计算而得到的.

    ```bash
    127.0.0.1:6379> pfcount mykey
    (integer) 10
    127.0.0.1:6379> pfcount yourkey
    (integer) 8
    127.0.0.1:6379> pfcount mykey yourkey
    (integer) 16
    ```

3. **pfmerge**：`PFMERGE destkey sourcekey [sourcekey ...]`

    将多个 HyperLogLog 合并（merge）为一个 HyperLogLog ， 合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集.

    合并得出的 HyperLogLog 会被储存在目标变量（第一个参数）里面， 如果该键并不存在， 那么命令在执行之前， 会先为该键创建一个空的.

    ```bash
    127.0.0.1:6379> pfmerge ourkey mykey yourkey
    OK
    127.0.0.1:6379> pfcount ourkey
    (integer) 16
    ```

如果允许容错(0.81% 标准错误)，那么一定可以使用 Hyperloglog

如果不允许容错，就使用 set 或者自己的数据类型即可 

### 3.3 Bitmap

**位存储** ：

统计用户信息，活跃，不活跃；登录、未登录；打卡，365打卡...两个状态的，都可以使用 Bitmap；

Bitmap 位图，数据结构，都是操作二进制位来进行记录，就只有 0 和 1 两个状态；

**使用：**

使用 bitmap 来记录周一到周日的打卡：周一：1；周二：0；周三：0；周四：1；...... 

```bash
127.0.0.1:6379> setbit sign 0 1
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 1
(integer) 0
127.0.0.1:6379> setbit sign 5 0
(integer) 0
127.0.0.1:6379> setbit sign 6 0
(integer) 0

# 查看某一天是否有打卡
127.0.0.1:6379> getbit sign 0
(integer) 1
127.0.0.1:6379> getbit sign 1
(integer) 0

# 统计操作，统计 打卡的天数
127.0.0.1:6379> bitcount sign  # 统计这周的打卡记录
(integer) 3
```

