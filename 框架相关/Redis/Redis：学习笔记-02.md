# Redis：学习笔记-02

> 根据 bilibili 上讲解 Redis 中观看较多的视频课程而顺带做的笔记，课程地址：[Redis最新超详细版教程通俗易懂](https://www.bilibili.com/video/BV1S54y1R7SB)，总计分为4篇，此为 2/4 篇。
>

## 4. 事物

Redis 事务本质：一组命令的集合；一个事务中的所有命令都会被序列化，在事务执行的过程中，会按照顺序执行。

一次性、顺序性、排他性的执行一系列命令。

```
------ 队列 set set set 执行------
```

Redis 事务没有没有隔离级别的概念。

所有的命令在事务中，并没有直接被执行，只有发起执行命令 exec 的时候才会执行。

**Redis单条命令式保存原子性的，但是事务不保证原子性**

redis的事务：

- 开启事务（multi）
- 命令入队（......）
- 执行事务（exec） 

### 4.1 正常执行事物

```bash
127.0.0.1:6379> multi		# 开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec		# 执行事务
1) OK
2) OK
3) "v1"
4) OK
```

### 4.2 放弃事物

```bash
127.0.0.1:6379> multi		# 开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> discard		# 取消事务
OK
127.0.0.1:6379> get k4		# 事务队列中命令都不会被执行
(nil)
```

### 4.3 编译型异常 

编译型异常（代码有问题，命令有错） ，事务中所有的命令都不会被执行 

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> getset k3		# 错误的命令
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> exec			# 执行事务报错
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k4			# 所有的命令都不会被执行
(nil)
```

### 4.4 运行时异常

运行时异常（如：1/0）， 如果事务队列中存在语法性，那么执行命令的时候，其他命令是可以正常执行的，错误命令抛出异常。

```bash
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr k1		# 会执行的时候失败
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> get k4
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range	# 虽然第一条命令报错了，但是依旧正常执行成功了
2) OK
3) "v4"
```

### 4.5 监控

**悲观锁：**很悲观，认为什么时候都会出问题，无论做什么都会加锁

**乐观锁：**很乐观，认为什么时候都不会出问题，所以不会上锁；更新数据的时候去判断一下，在此期间是否有人修改过这个数据，获取version更新的时候比较 version

**示例：**

- 正常执行成功

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money	# 监视 money 对象
OK
127.0.0.1:6379> multi		# 事务正常结束，数据期间没有发生变动，这个时候就正常执行成功
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> incrby out 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20
```

测试多线程修改值 , 使用watch 可以当做Redis的乐观锁操作 

```bash
127.0.0.1:6379> watch money		# 监视 money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 50
QUEUED
127.0.0.1:6379> incrby out 50
QUEUED
# -----
# 此时打开另一个client对money进行修改：set money 1000
# -----
127.0.0.1:6379> exec  # 执行之前，另外一个线程，修改了我们的值，这个时候，就会导致事务执行失败
(nil)
127.0.0.1:6379> get money  # 可以看到上方事物执行失败，而money的值已经在另一个线程中被修改了
"1000"
```

如果修改失败，获取最新的值就好

```bash
127.0.0.1:6379> unwatch  	# 如果发现事物执行失败，就先解锁
OK
127.0.0.1:6379> watch money	# 获取最新的值，再次监视
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 50
QUEUED
127.0.0.1:6379> incrby out 50
QUEUED
127.0.0.1:6379> exec  # 比对监视的值是否发生了变化，若没有变化，则执行成功，如果变化则执行失败
1) (integer) 950
2) (integer) 70
```

## 5. Jedis

### 5.1 连接测试

我们要使用 Java 来操作 Redis 

> Jedis 是 Redis 官方推荐的 Java 连接开发工具，使用 Java 操作 Redis 中间件，如果你要使用 Java 操作Redis，那么一定要对 Jedis 十分的熟悉 

1. 创建Maven项目并导入相关依赖：

   ```xml
   <dependencies>
       <dependency>
           <groupId>redis.clients</groupId>
           <artifactId>jedis</artifactId>
           <version>3.2.0</version>
       </dependency>
   
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
           <version>1.2.62</version>
       </dependency>
   
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.12</version>
       </dependency>
   </dependencies>
   ```

2. 编码测试：

   - 连接数据库
   - 操作命令
   - 断开连接

   ```java
   @Test
   public void test(){
       // new Jedis 对象即可
       Jedis jedis = new Jedis("127.0.0.1", 6379);
       System.out.println(jedis.ping());
   }
   ```

   输出：`PONG`

### 5.2 常用API

**key测试：**

```java
@Test
public void testKey(){
    Jedis jedis = new Jedis("127.0.0.1", 6379);

    System.out.println("清空数据："+jedis.flushDB());  // OK
    System.out.println("判断某个键是否存在："+jedis.exists("username"));  // false
    System.out.println("新增<'username','kuangshen'>的键值对："+jedis.set("username", "kuangshen"));  // OK
    System.out.println("新增<'password','password'>的键值对："+jedis.set("password", "password"));  // OK
    System.out.print("系统中所有的键如下：");
    Set<String> keys = jedis.keys("*"); 
    System.out.println(keys);  // [password, username]
    System.out.println("删除键password:"+jedis.del("password"));  // 1
    System.out.println("判断键password是否存在："+jedis.exists("password"));  // false
    System.out.println("查看键username所存储的值的类型："+jedis.type("username"));  // string
    System.out.println("随机返回key空间的一个："+jedis.randomKey());  // username
    System.out.println("重命名key："+jedis.rename("username","name"));  // OK
    System.out.println("取出改后的name："+jedis.get("name"));  // kuangshen
    System.out.println("按索引查询："+jedis.select(0));  // OK
    System.out.println("删除当前选择数据库中的所有key："+jedis.flushDB());  // OK
    System.out.println("返回当前数据库中key的数目："+jedis.dbSize());  // 0
    System.out.println("删除所有数据库中的所有key："+jedis.flushAll());  // OK
}
```

**String测试**：

```java
@Test
public void testString(){
    Jedis jedis = new Jedis("127.0.0.1", 6379);

    jedis.flushDB();
    System.out.println("===========增加数据===========");
    System.out.println(jedis.set("key1","value1"));  // OK
    System.out.println(jedis.set("key2","value2"));  // OK
    System.out.println(jedis.set("key3", "value3"));  // OK
    System.out.println("删除键key2:"+jedis.del("key2"));  // 1
    System.out.println("获取键key2:"+jedis.get("key2"));  // null
    System.out.println("修改key1:"+jedis.set("key1", "value1Changed"));  // ok
    System.out.println("获取key1的值："+jedis.get("key1"));  // value1Changed
    System.out.println("在key3后面加入值："+jedis.append("key3", "End"));  // 9
    System.out.println("key3的值："+jedis.get("key3"));  // value3End
    System.out.println("增加多个键值对："+jedis.mset("key01","value01","key02","value02","key03","value03"));  // OK
    System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03"));  // [value01, value02, value03]
    System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03","key04"));  // [value01, value02, value03, null]
    System.out.println("删除多个键值对："+jedis.del("key01","key02"));  // 2
    System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03"));  // [null, null, value03]

    jedis.flushDB();
    System.out.println("===========新增键值对防止覆盖原先值==============");
    System.out.println(jedis.setnx("key1", "value1"));  // 1
    System.out.println(jedis.setnx("key2", "value2"));  // 1
    System.out.println(jedis.setnx("key2", "value2-new"));  // 0
    System.out.println(jedis.get("key1"));  // value1
    System.out.println(jedis.get("key2"));  // value2

    System.out.println("===========新增键值对并设置有效时间=============");
    System.out.println(jedis.setex("key3", 2, "value3"));  // OK
    System.out.println(jedis.get("key3"));  // value3
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(jedis.get("key3"));  // null

    System.out.println("===========获取原值，更新为新值==========");
    System.out.println(jedis.getSet("key2", "key2GetSet"));  // value2
    System.out.println(jedis.get("key2"));  // key2GetSet

    System.out.println("获得key2的值的字串："+jedis.getrange("key2", 2, 4));  // y2G
}
```

**List测试：**

```java
@Test
public void testList(){
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    jedis.flushDB();
    System.out.println("===========添加一个list===========");
    jedis.lpush("collections", "ArrayList", "Vector", "Stack", "HashMap", "WeakHashMap", "LinkedHashMap");
    jedis.lpush("collections", "HashSet");
    jedis.lpush("collections", "TreeSet");
    jedis.lpush("collections", "TreeMap");
    System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));//-1代表倒数第一个元素，-2代表倒数第二个元素,end为-1表示查询全部
    // 输出：[TreeMap, TreeSet, HashSet, LinkedHashMap, WeakHashMap, HashMap, Stack, Vector, ArrayList]
    System.out.println("collections区间0-3的元素："+jedis.lrange("collections",0,3));  // [TreeMap, TreeSet, HashSet, LinkedHashMap]
    System.out.println("===============================");
    // 删除列表指定的值 ，第二个参数为删除的个数（有重复时），后add进去的值先被删，类似于出栈
    System.out.println("删除指定元素个数："+jedis.lrem("collections", 2, "HashMap"));  // 1
    System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));  // [TreeMap, TreeSet, HashSet, LinkedHashMap, WeakHashMap, Stack, Vector, ArrayList]
    System.out.println("删除下表0-3区间之外的元素："+jedis.ltrim("collections", 0, 3));  // OK
    System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));  // [TreeMap, TreeSet, HashSet, LinkedHashMap]
    System.out.println("collections列表出栈（左端）："+jedis.lpop("collections"));  // TreeMap
    System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));  // [TreeSet, HashSet, LinkedHashMap]
    System.out.println("collections添加元素，从列表右端，与lpush相对应："+jedis.rpush("collections", "EnumMap"));  // 4
    System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));  // [TreeSet, HashSet, LinkedHashMap, EnumMap]
    System.out.println("collections列表出栈（右端）："+jedis.rpop("collections"));  // EnumMap
    System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));  // [TreeSet, HashSet, LinkedHashMap]
    System.out.println("修改collections指定下标1的内容："+jedis.lset("collections", 1, "LinkedArrayList"));  // OK
    System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));  // [TreeSet, LinkedArrayList, LinkedHashMap]
    System.out.println("===============================");
    System.out.println("collections的长度："+jedis.llen("collections"));  // 3
    System.out.println("获取collections下标为2的元素："+jedis.lindex("collections", 2));  // LinkedHashMap
    System.out.println("===============================");
    jedis.lpush("sortedList", "3","6","2","0","7","4"); 
    System.out.println("sortedList排序前："+jedis.lrange("sortedList", 0, -1));  // [4, 7, 0, 2, 6, 3]
    System.out.println(jedis.sort("sortedList"));  // [0, 2, 3, 4, 6, 7]
    System.out.println("sortedList排序后："+jedis.lrange("sortedList", 0, -1));  // [4, 7, 0, 2, 6, 3]
}
```

**set测试**：

```java
@Test
public void testSet(){
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    jedis.flushDB();
    System.out.println("============向集合中添加元素（不重复）============");
    System.out.println(jedis.sadd("eleSet", "e1","e2","e4","e3","e0","e8","e7","e5"));  // 8
    System.out.println(jedis.sadd("eleSet", "e6"));  // 1
    System.out.println(jedis.sadd("eleSet", "e6"));  // 0
    System.out.println("eleSet的所有元素为："+jedis.smembers("eleSet"));  // [e4, e2, e0, e5, e3, e8, e1, e7, e6]
    System.out.println("删除一个元素e0："+jedis.srem("eleSet", "e0"));  // 1
    System.out.println("eleSet的所有元素为："+jedis.smembers("eleSet"));  // [e2, e5, e3, e8, e1, e7, e6, e4]
    System.out.println("删除两个元素e7和e6："+jedis.srem("eleSet", "e7","e6"));  // 2
    System.out.println("eleSet的所有元素为："+jedis.smembers("eleSet"));  // [e3, e8, e1, e4, e2, e5]
    System.out.println("随机的移除集合中的一个元素："+jedis.spop("eleSet"));  // e4
    System.out.println("随机的移除集合中的一个元素："+jedis.spop("eleSet"));  // e2
    System.out.println("eleSet的所有元素为："+jedis.smembers("eleSet"));  // [e8, e1, e5, e3]
    System.out.println("eleSet中包含元素的个数："+jedis.scard("eleSet"));  // 4
    System.out.println("e3是否在eleSet中："+jedis.sismember("eleSet", "e3"));  // true
    System.out.println("e1是否在eleSet中："+jedis.sismember("eleSet", "e1"));  // true
    System.out.println("e1是否在eleSet中："+jedis.sismember("eleSet", "e5"));  // true
    System.out.println("=================================");
    System.out.println(jedis.sadd("eleSet1", "e1","e2","e4","e3","e0","e8","e7","e5"));  // 8
    System.out.println(jedis.sadd("eleSet2", "e1","e2","e4","e3","e0","e8"));  // 6
    System.out.println("将eleSet1中删除e1并存入eleSet3中："+jedis.smove("eleSet1", "eleSet3", "e1"));  //移到集合元素:1
    System.out.println("将eleSet1中删除e2并存入eleSet3中："+jedis.smove("eleSet1", "eleSet3", "e2"));  // 1
    System.out.println("eleSet1中的元素："+jedis.smembers("eleSet1"));  // [e7, e8, e4, e0, e5, e3]
    System.out.println("eleSet3中的元素："+jedis.smembers("eleSet3"));  // [e2, e1]
    System.out.println("============集合运算=================");
    System.out.println("eleSet1中的元素："+jedis.smembers("eleSet1"));  // [e7, e8, e4, e0, e5, e3]
    System.out.println("eleSet2中的元素："+jedis.smembers("eleSet2"));  // [e2, e3, e1, e8, e4, e0]
    System.out.println("eleSet1和eleSet2的交集:"+jedis.sinter("eleSet1","eleSet2"));  // [e3, e8, e4, e0]
    System.out.println("eleSet1和eleSet2的并集:"+jedis.sunion("eleSet1","eleSet2"));  // [e2, e0, e5, e3, e8, e1, e7, e4]
    System.out.println("eleSet1和eleSet2的差集:"+jedis.sdiff("eleSet1","eleSet2"));  // [e5, e7]
    jedis.sinterstore("eleSet4","eleSet1","eleSet2");  // 求交集并将交集保存到dstkey的集合
    System.out.println("eleSet4中的元素："+jedis.smembers("eleSet4"));  // [e4, e0, e3, e8]
}
```

**hash测试：**

```java
@Test
public void testHash(){
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    jedis.flushDB();
    Map<String,String> map = new HashMap<String,String>();
    map.put("key1","value1");
    map.put("key2","value2");
    map.put("key3","value3");
    map.put("key4","value4");
    // 添加名称为hash（key）的hash元素
    jedis.hmset("hash",map); 
    // 向名称为hash的hash中添加key为key5，value为value5元素
    jedis.hset("hash", "key5", "value5");
    System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));//  {key1=value1, key2=value2, key5=value5, key3=value3, key4=value4}
    System.out.println("散列hash的所有键为："+jedis.hkeys("hash"));// [key1, key2, key5, key3, key4]
    System.out.println("散列hash的所有值为："+jedis.hvals("hash"));// [value3, value1, value2, value4, value5]
    System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6："+jedis.hincrBy("hash", "key6", 6));  // 6
    System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));  // {key1=value1, key2=value2, key5=value5, key6=6, key3=value3, key4=value4}
    System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6："+jedis.hincrBy("hash", "key6", 3));  // 9
    System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));  // {key1=value1, key2=value2, key5=value5, key6=9, key3=value3, key4=value4}
    System.out.println("删除一个或者多个键值对："+jedis.hdel("hash", "key2"));  // 1
    System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));  // {key1=value1, key5=value5, key6=9, key3=value3, key4=value4}
    System.out.println("散列hash中键值对的个数："+jedis.hlen("hash"));  // 5
    System.out.println("判断hash中是否存在key2："+jedis.hexists("hash","key2"));  // false
    System.out.println("判断hash中是否存在key3："+jedis.hexists("hash","key3"));  // true
    System.out.println("获取hash中的值："+jedis.hmget("hash","key3"));  // [value3]
    System.out.println("获取hash中的值："+jedis.hmget("hash","key3","key4"));  // [value3, value4]
}
```

### 5.3 事物操作

```java
@Test
public void test(){
    Jedis jedis = new Jedis("127.0.0.1", 6379);

    jedis.flushDB();

    JSONObject jsonObject = new JSONObject();
    jsonObject.put("hello", "world");
    jsonObject.put("hi", "redis");

    // 开启事务
    Transaction multi = jedis.multi();
    String result = jsonObject.toJSONString();
    try {
        multi.set("user1", result);
        multi.set("user2", result);
		int i = 1/0;   // 代码抛出异常事务，执行失败
        multi.exec();  // 执行事务
    }catch (Exception e){
        multi.discard();  // 放弃事务
        e.printStackTrace();
    }finally {
        System.out.println(jedis.get("user1"));
        System.out.println(jedis.get("user2"));
        jedis.close();  // 关闭连接
    }
}
```

1. 当未出现异常时，输出：

```java
{"hi":"redis","hello":"world"}
{"hi":"redis","hello":"world"}
```

2. 当出现异常时，输出：

```java
java.lang.ArithmeticException: / by zero
	at cn.xyc.JedisTx.test(JedisTx.java:26)
null
null

```

## 6. SpringBoot 整合

### 6.1 概述

SpringBoot 操作数据：`spring-data：jpa`、`jdbc`、`mongodb`、`Redis`

SpringData 也是和 SpringBoot 齐名的项目

> 说明： 在 SpringBoot2.x 之后，原来使用的 jedis 被替换为了 lettuce
>
> jedis : 采用的直连，多个线程操作的话，是不安全的，如果想要避免不安全的，使用 jedis pool 连接池，更像 BIO 模式
>
> lettuce : 采用 netty，实例可以在多个线程中进行共享，不存在线程不安全的情况，可以减少线程数据了，更像 NIO 模式  

**通过 springboot 对 redis 进行整合：**

1. 导入依赖：

```xml
<!-- 操作redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. 连接配置

```properties
# 配置redis：其实都是默认的
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

3. 测试

```java
@SpringBootTest
public class SpringbootRedisApplicattionTests {

    @Autowired
    // private RedisTemplate<String, String> redisTemplate;
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads(){
        // redisTemplate 操作不同的数据类型，api和我们的指令是一样的
        // opsForValue 操作字符串 类似String
        // opsForList  操作List 类似List
        // opsForSet   操作Set
        // opsForHash  操作Hash
        // opsForZSet  操作zset
        // opsForGeo   操作geospatial
        // opsForHyperLogLog  操作HyperLogLog
        redisTemplate.opsForValue().set("name", "zhangsan");
        System.out.println(redisTemplate.opsForValue().get("name"));
    }

    @Test
    void test(){
        // 除了基本的操作，我们常用的方法都可以直接通过redisTemplate操作，比如事务，和基本的CRUD
        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
        connection.flushDb();
        connection.close();
    }
}

```

### 6.2 源码分析

也是通过 `RedisAutoConfiguration` 进行了自动配置

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
// 在RedisProperties.class中定义的相关的属性
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")  // 通过自己定义redisTemplate可以覆盖该类的创建
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
        // 默认的 RedisTemplate 没有过多的设置，redis 对象都是需要序列化
        // 两个泛型都是 Object, Object 的类型，我们使用需要强制转换 <String, Object>
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
    // 由于 String 是redis中最常使用的类型，所以说单独提出来了一个bean
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}
}
```

查看`RedisTemplate`类：

```java
public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {
    private boolean enableTransactionSupport = false;
    private boolean exposeConnection = false;
    private boolean initialized = false;
    private boolean enableDefaultSerializer = true;
    private @Nullable RedisSerializer<?> defaultSerializer;
    private @Nullable ClassLoader classLoader;

    // 有关序列化的配置
    @SuppressWarnings("rawtypes") private @Nullable RedisSerializer keySerializer = null;
    @SuppressWarnings("rawtypes") private @Nullable RedisSerializer valueSerializer = null;
    @SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashKeySerializer = null;
    @SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashValueSerializer = null;
    private RedisSerializer<String> stringSerializer = RedisSerializer.string();
    
    @Override
    public void afterPropertiesSet() {

        super.afterPropertiesSet();

        boolean defaultUsed = false;

        if (defaultSerializer == null) {
			// 默认的序列化方式就是使用jdk序列化
            defaultSerializer = new JdkSerializationRedisSerializer(
                classLoader != null ? classLoader : this.getClass().getClassLoader());
        }

        if (enableDefaultSerializer) {

            if (keySerializer == null) {
                keySerializer = defaultSerializer;
                defaultUsed = true;
            }
            if (valueSerializer == null) {
                valueSerializer = defaultSerializer;
                defaultUsed = true;
            }
            if (hashKeySerializer == null) {
                hashKeySerializer = defaultSerializer;
                defaultUsed = true;
            }
            if (hashValueSerializer == null) {
                hashValueSerializer = defaultSerializer;
                defaultUsed = true;
            }
        }

        if (enableDefaultSerializer && defaultUsed) {
            Assert.notNull(defaultSerializer, "default serializer null and not all serializers initialized");
        }

        if (scriptExecutor == null) {
            this.scriptExecutor = new DefaultScriptExecutor<>(this);
        }

        initialized = true;
    }
}
```

使用默认的JDK编码方式进行对象的保存：

> 创建对象：
>
> ```java
> public class User implements Serializable{  // 想要保存对象必须实现Serializable接口
>     private String name;
>     private Integer age;
> }
> ```

```java
@Test
void testJdkSerializer(){
    User user = new User("zhangsan", 18);
    redisTemplate.opsForValue().set("user", user);
    System.out.println(redisTemplate.opsForValue().get("user"));
}
```

输出：

```bash
控制台输出：User{name='zhangsan', age=18}

通过redis命令控制终端查询：
127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x04user"
127.0.0.1:6379> get "\xac\xed\x00\x05t\x00\x04user"
"\xac\xed\x00\x05sr\x00\"cn.xyc.springbootredis.domain.User\xaak\x8a\xca\xa1\a\xf1\n\x02\x00\x02L\x00\x03aget\x00\x13Ljava/lang/Integer;L\x00\x04namet\x00\x12Ljava/lang/String;xpsr\x00\x11java.lang.Integer\x12\xe2\xa0\xa4\xf7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00\x00\x00\x12t\x00\bzhangsan"
```

通过自定义一个 `RedisTemplete`，使用 json 的方式进行序列化对象

```java
@Configuration
public class RedisConfig {

    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory){
        // 我们为了自己开发方便，一般直接使用 <String, Object>
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        // Json序列化配置
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        // String 的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        template.afterPropertiesSet();

        return template;
    }
}
```

输出：

```bash
控制台输出：User{name='zhangsan', age=18}

通过redis命令控制终端查询：
127.0.0.1:6379> get user
"[\"cn.xyc.springbootredis.domain.User\",{\"name\":\"zhangsan\",\"age\":18}]"
```

> 自定义工具类操作Redis：
>
> ```java
> package cn.xyc.springbootredis.utils;
> 
> import org.springframework.beans.factory.annotation.Autowired;
> import org.springframework.data.redis.core.RedisTemplate;
> import org.springframework.stereotype.Component;
> import org.springframework.util.CollectionUtils;
> 
> import java.util.List;
> import java.util.Map;
> import java.util.Set;
> import java.util.concurrent.TimeUnit;
> 
> @Component
> public final class RedisUtil {
> 
>     @Autowired
>     private RedisTemplate<String, Object> redisTemplate;
> 
>     // =============================common============================
>     	/**
>      * 指定缓存失效时间
>      * @param key  键
>      * @param time 时间(秒)
>   	 */
>     public boolean expire(String key, long time) {
>         try {
>             if (time > 0) {
>                 redisTemplate.expire(key, time, TimeUnit.SECONDS);
>             }
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
>     /**
>      * 根据key 获取过期时间
>      * @param key 键 不能为null
>      * @return 时间(秒) 返回0代表为永久有效
>   	 */
>     public long getExpire(String key) {
>         return redisTemplate.getExpire(key, TimeUnit.SECONDS);
>     }
> 
> 
>     /**
>      * 判断key是否存在
>      * @param key 键
>      * @return true 存在 false不存在
>   	 */
>     public boolean hasKey(String key) {
>         try {
>             return redisTemplate.hasKey(key);
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 删除缓存
>      * @param key 可以传一个值 或多个
>   	 */
>     @SuppressWarnings("unchecked")
>     public void del(String... key) {
>         if (key != null && key.length > 0) {
>             if (key.length == 1) {
>                 redisTemplate.delete(key[0]);
>             } else {
>                 redisTemplate.delete(CollectionUtils.arrayToList(key));
>             }
>         }
>     }
> 
> 
>     // ============================String=============================
> 
>     /**
>      * 普通缓存获取
>      * @param key 键
>      * @return 值
>   	 */
>     public Object get(String key) {
>         return key == null ? null : redisTemplate.opsForValue().get(key);
>     }
> 
>     /**
>      * 普通缓存放入
>      * @param key   键
>      * @param value 值
>      * @return true成功 false失败
>   	 */
> 
>     public boolean set(String key, Object value) {
>         try {
>             redisTemplate.opsForValue().set(key, value);
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 普通缓存放入并设置时间
>      * @param key   键
>      * @param value 值
>      * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
>      * @return true成功 false 失败
>   	 */
> 
>     public boolean set(String key, Object value, long time) {
>         try {
>             if (time > 0) {
>                 redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
>             } else {
>                 set(key, value);
>             }
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 递增
>      * @param key   键
>      * @param delta 要增加几(大于0)
>   	 */
>     public long incr(String key, long delta) {
>         if (delta < 0) {
>             throw new RuntimeException("递增因子必须大于0");
>         }
>         return redisTemplate.opsForValue().increment(key, delta);
>     }
> 
> 
>     /**
>      * 递减
>      * @param key   键
>      * @param delta 要减少几(小于0)
>   	 */
>     public long decr(String key, long delta) {
>         if (delta < 0) {
>             throw new RuntimeException("递减因子必须大于0");
>         }
>         return redisTemplate.opsForValue().increment(key, -delta);
>     }
> 
> 
>     // ================================Map=================================
> 
>     /**
>      * HashGet
>      * @param key  键 不能为null
>      * @param item 项 不能为null
>   	 */
>     public Object hget(String key, String item) {
>         return redisTemplate.opsForHash().get(key, item);
>     }
> 
>     /**
>      * 获取hashKey对应的所有键值
>      * @param key 键
>      * @return 对应的多个键值
>   	 */
>     public Map<Object, Object> hmget(String key) {
>         return redisTemplate.opsForHash().entries(key);
>     }
> 
>     /**
>      * HashSet
>      * @param key 键
>      * @param map 对应多个键值
>   	 */
>     public boolean hmset(String key, Map<String, Object> map) {
>         try {
>             redisTemplate.opsForHash().putAll(key, map);
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * HashSet 并设置时间
>      * @param key  键
>      * @param map  对应多个键值
>      * @param time 时间(秒)
>      * @return true成功 false失败
>   	 */
>     public boolean hmset(String key, Map<String, Object> map, long time) {
>         try {
>             redisTemplate.opsForHash().putAll(key, map);
>             if (time > 0) {
>                 expire(key, time);
>             }
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 向一张hash表中放入数据,如果不存在将创建
>      * @param key   键
>      * @param item  项
>      * @param value 值
>      * @return true 成功 false失败
>   	 */
>     public boolean hset(String key, String item, Object value) {
>         try {
>             redisTemplate.opsForHash().put(key, item, value);
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
>     /**
>      * 向一张hash表中放入数据,如果不存在将创建
>      * @param key   键
>      * @param item  项
>      * @param value 值
>      * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
>      * @return true 成功 false失败
>   	 */
>     public boolean hset(String key, String item, Object value, long time) {
>         try {
>             redisTemplate.opsForHash().put(key, item, value);
>             if (time > 0) {
>                 expire(key, time);
>             }
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 删除hash表中的值
>      * @param key  键 不能为null
>      * @param item 项 可以使多个 不能为null
>   	 */
>     public void hdel(String key, Object... item) {
>         redisTemplate.opsForHash().delete(key, item);
>     }
> 
> 
>     /**
>      * 判断hash表中是否有该项的值
>      * @param key  键 不能为null
>      * @param item 项 不能为null
>      * @return true 存在 false不存在
>   	 */
>     public boolean hHasKey(String key, String item) {
>         return redisTemplate.opsForHash().hasKey(key, item);
>     }
> 
> 
>     /**
>      * hash递增 如果不存在,就会创建一个 并把新增后的值返回
>      * @param key  键
>      * @param item 项
>      * @param by   要增加几(大于0)
>   	 */
>     public double hincr(String key, String item, double by) {
>         return redisTemplate.opsForHash().increment(key, item, by);
>     }
> 
> 
>     /**
>      * hash递减
>      * @param key  键
>      * @param item 项
>      * @param by   要减少记(小于0)
>   	 */
>     public double hdecr(String key, String item, double by) {
>         return redisTemplate.opsForHash().increment(key, item, -by);
>     }
> 
> 
>     // ============================set=============================
> 
>     /**
>      * 根据key获取Set中的所有值
>      * @param key 键
>   	 */
>     public Set<Object> sGet(String key) {
>         try {
>             return redisTemplate.opsForSet().members(key);
>         } catch (Exception e) {
>             e.printStackTrace();
>             return null;
>         }
>     }
> 
> 
>     /**
>      * 根据value从一个set中查询,是否存在
>      * @param key   键
>      * @param value 值
>      * @return true 存在 false不存在
>   	 */
>     public boolean sHasKey(String key, Object value) {
>         try {
>             return redisTemplate.opsForSet().isMember(key, value);
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 将数据放入set缓存
>      * @param key    键
>      * @param values 值 可以是多个
>      * @return 成功个数
>   	 */
>     public long sSet(String key, Object... values) {
>         try {
>             return redisTemplate.opsForSet().add(key, values);
>         } catch (Exception e) {
>             e.printStackTrace();
>             return 0;
>         }
>     }
> 
> 
>     /**
>      * 将set数据放入缓存
>      * @param key    键
>      * @param time   时间(秒)
>      * @param values 值 可以是多个
>      * @return 成功个数
>   	 */
>     public long sSetAndTime(String key, long time, Object... values) {
>         try {
>             Long count = redisTemplate.opsForSet().add(key, values);
>             if (time > 0)
>                 expire(key, time);
>             return count;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return 0;
>         }
>     }
> 
> 
>     /**
>      * 获取set缓存的长度
>      * @param key 键
>   	 */
>     public long sGetSetSize(String key) {
>         try {
>             return redisTemplate.opsForSet().size(key);
>         } catch (Exception e) {
>             e.printStackTrace();
>             return 0;
>         }
>     }
> 
> 
>     /**
>      * 移除值为value的
>      * @param key    键
>      * @param values 值 可以是多个
>      * @return 移除的个数
>  	 */
> 
>     public long setRemove(String key, Object... values) {
>         try {
>             Long count = redisTemplate.opsForSet().remove(key, values);
>             return count;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return 0;
>         }
>     }
> 
>     // ===============================list=================================
> 
>     /**
>      * 获取list缓存的内容
>      * @param key   键
>      * @param start 开始
>      * @param end   结束 0 到 -1代表所有值
>   	 */
>     public List<Object> lGet(String key, long start, long end) {
>         try {
>             return redisTemplate.opsForList().range(key, start, end);
>         } catch (Exception e) {
>             e.printStackTrace();
>             return null;
>         }
>     }
> 
> 
>     /**
>      * 获取list缓存的长度
>      * @param key 键
>      */
>     public long lGetListSize(String key) {
>         try {
>             return redisTemplate.opsForList().size(key);
>         } catch (Exception e) {
>             e.printStackTrace();
>             return 0;
>         }
>     }
> 
> 
>     /**
>      * 通过索引 获取list中的值
>      * @param key   键
>      * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
>   	 */
>     public Object lGetIndex(String key, long index) {
>         try {
>             return redisTemplate.opsForList().index(key, index);
>         } catch (Exception e) {
>             e.printStackTrace();
>             return null;
>         }
>     }
> 
> 
>     /**
>      * 将list放入缓存
>      * @param key   键
>      * @param value 值
>   	 */
>     public boolean lSet(String key, Object value) {
>         try {
>             redisTemplate.opsForList().rightPush(key, value);
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 将list放入缓存
>      * @param key   键
>      * @param value 值
>      * @param time  时间(秒)
>   	 */
>     public boolean lSet(String key, Object value, long time) {
>         try {
>             redisTemplate.opsForList().rightPush(key, value);
>             if (time > 0)
>                 expire(key, time);
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
> 
>     }
> 
> 
>     /**
>      * 将list放入缓存
>      * @param key   键
>      * @param value 值
>      * @return
>   	 */
>     public boolean lSet(String key, List<Object> value) {
>         try {
>             redisTemplate.opsForList().rightPushAll(key, value);
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
> 
>     }
> 
> 
>     /**
>      * 将list放入缓存
>      * @param key   键
>      * @param value 值
>      * @param time  时间(秒)
>      * @return
>   	 */
>     public boolean lSet(String key, List<Object> value, long time) {
>         try {
>             redisTemplate.opsForList().rightPushAll(key, value);
>             if (time > 0)
>                 expire(key, time);
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 根据索引修改list中的某条数据
>      * @param key   键
>      * @param index 索引
>      * @param value 值
>      * @return
>      */
> 
>     public boolean lUpdateIndex(String key, long index, Object value) {
>         try {
>             redisTemplate.opsForList().set(key, index, value);
>             return true;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return false;
>         }
>     }
> 
> 
>     /**
>      * 移除N个值为value
>      * @param key   键
>      * @param count 移除多少个
>      * @param value 值
>      * @return 移除的个数
>   */
> 
>     public long lRemove(String key, long count, Object value) {
>         try {
>             Long remove = redisTemplate.opsForList().remove(key, count, value);
>             return remove;
>         } catch (Exception e) {
>             e.printStackTrace();
>             return 0;
>         }
>     }
> }
> ```
> 
> 测试使用：
> 
> ```java
> @Autowired
> private RedisUtil redisUtil;
> 
> @Test
> void testUtils(){
>     redisUtil.lSet("list", 1);
>     redisUtil.lSet("list", 2);
>     redisUtil.lSet("list", Lists.list(3, 4));
>     List<Object> list = redisUtil.lGet("list", 0, -1);
>     System.out.println(list.toString());  // [1, 2, 3, 4]
> }
> ```



