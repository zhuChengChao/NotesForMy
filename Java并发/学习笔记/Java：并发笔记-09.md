# Java：并发笔记-09

> 说明：观看了 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444/)** 的课程 [java并发编程](https://www.bilibili.com/video/BV16J411h7Rd) 后所作的学习笔记，总计拆分为9篇笔记，此为9/9。

## 7. 共享模型之工具-2

### 原理：AQS 原理

> 对于 AQS 的原理这部分内容，没很好的理解，等功力深厚了再回来好好理解一下，笔记也就不贴出来了

### 原理：ReentrantLock 原理

> 同样对于 ReentrantLock 的原理这部分内容，没很好的理解，等功力深厚了再回来好好理解一下

### 7.2 J.U.C

#### 3. 读写锁

##### 3.1 ReentrantReadWriteLock 

当读操作远远高于写操作时，这时候使用 **读写锁** 让 **读-读** 可以并发，提高性能。 类似于数据库中的 `select ... from ... lock in share mode;`

提供一个 数据容器类 内部分别使用读锁保护数据的 `read()` 方法，写锁保护数据的 `write()` 方法 

```java
class DataContainer{

    private Object data;
    // 读写锁
    private ReentrantReadWriteLock rw = new ReentrantReadWriteLock();
    // 读锁
    private ReentrantReadWriteLock.ReadLock r = rw.readLock();
    // 写锁
    private ReentrantReadWriteLock.WriteLock w = rw.writeLock();

    public Object read(){
        LoggerUtils.LOGGER.debug("获取读锁...");
        r.lock();
        try {
            LoggerUtils.LOGGER.debug("读取数据");
            Sleeper.sleep(1);
            return data;
        }finally {
            LoggerUtils.LOGGER.debug("释放读锁...");
            r.unlock();
        }
    }

    public void write(){
        LoggerUtils.LOGGER.debug("获取写锁...");
        w.lock();
        try {
            LoggerUtils.LOGGER.debug("写入数据");
            Sleeper.sleep(1);
        }finally {
            LoggerUtils.LOGGER.debug("释放写锁...");
            w.unlock();
        }
    }
}
```

测试 **读锁-读锁** 可以并发 

```java
DataContainer dataContainer = new DataContainer();
new Thread(()->{
    dataContainer.read();
}, "t1").start();

new Thread(()->{
    dataContainer.read();
}, "t2").start();
```

输出结果，从这里可以看到 Thread-0 锁定期间，Thread-1 的读操作不受影响 

```
14:40:04.638 cn.util.LoggerUtils [t2] - 获取读锁...
14:40:04.638 cn.util.LoggerUtils [t1] - 获取读锁...
14:40:04.640 cn.util.LoggerUtils [t2] - 读取数据
14:40:04.640 cn.util.LoggerUtils [t1] - 读取数据
14:40:05.642 cn.util.LoggerUtils [t2] - 释放读锁...
14:40:05.642 cn.util.LoggerUtils [t1] - 释放读锁...
```

测试 **读锁-写锁** 相互阻塞 

```java
DataContainer dataContainer = new DataContainer();
new Thread(()->{
    dataContainer.read();
}, "t1").start();

new Thread(()->{
    dataContainer.write();
}, "t2").start();
```

输出结果 

```
14:41:08.246 cn.util.LoggerUtils [t1] - 获取读锁...
14:41:08.246 cn.util.LoggerUtils [t2] - 获取写锁...
14:41:08.249 cn.util.LoggerUtils [t1] - 读取数据
14:41:09.251 cn.util.LoggerUtils [t1] - 释放读锁...
14:41:09.251 cn.util.LoggerUtils [t2] - 写入数据
14:41:10.252 cn.util.LoggerUtils [t2] - 释放写锁...
```

**写锁-写锁** 也是相互阻塞的，这里就不测试了 

**注意事项**

- 读锁不支持条件变量，写锁支持条件变量

- **重入时升级不支持**：即持有读锁的情况下去获取写锁，会导致获取写锁永久等待 

  ```java
  r.lock();  // 获取读锁
  try {
      // ...
      w.lock();  // 有读锁的情况下去获取写作，会导致写锁永久等待
      try {
          // ...
      } finally{
          w.unlock();
      }
  } finally{
      r.unlock();
  }
  ```

- **重入时降级支持**：即持有写锁的情况下去获取读锁 

  ```java
  class CachedData {
      
      Object data;  // 需要缓存的数据
      // 是否有效，如果失效，需要重新计算 data  
      volatile boolean cacheValid;  // 数据是否有效
      final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
      
      void processCachedData() {
          rwl.readLock().lock();  // 加一个读锁
          if (!cacheValid) {      // 数据已经失效
              // 获取写锁前必须释放读锁，因为不支持升级
              rwl.readLock().unlock();
              rwl.writeLock().lock();
              try {
                  // 判断是否有其它线程已经获取了写锁、更新了缓存, 避免重复更新
                  if (!cacheValid) {
                      data = ...
                      cacheValid = true;
                  }
                  // 降级为读锁, 释放写锁, 这样能够让其它线程读取缓存
                  rwl.readLock().lock();
              } finally {
                  rwl.writeLock().unlock();
              }
          }
          // 自己用完数据, 释放读锁
          try {
              use(data);
          } finally {
              rwl.readLock().unlock();
          }
      }
  }
  ```

##### 应用：缓存

> #### 1. 缓存更新策略 
>
> **更新时，是先清缓存还是先更新数据库**
>
> * 先清缓存
>
> ![缓存](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251549635.PNG)
>
> * 先更新数据库 
>
> ![缓存2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251549640.PNG)
>
> * 补充一种情况，假设查询线程 A 查询数据时恰好缓存数据由于时间到期失效，或是第一次查询 
>
> ![缓存3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251549157.PNG)
>
> > 这种情况的出现几率非常小，见 facebook 论文 
>
> #### 2. 读写锁实现一致性缓存 
>
> 使用读写锁实现一个简单的按需加载缓存 
>
> ```java
> class GenericCachedDao<T> {
>     // HashMap 作为缓存非线程安全, 需要保护
>     HashMap<SqlPair, T> map = new HashMap<>();
>     ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
>     GenericDao genericDao = new GenericDao();
> 
>     public int update(String sql, Object... params) {
>         SqlPair key = new SqlPair(sql, params);
>         // 加写锁, 防止其它线程对缓存读取和更改
>         lock.writeLock().lock();
>         try {
>             int rows = genericDao.update(sql, params);
>             map.clear();  // 清空缓存
>             return rows;
>         } finally {
>             lock.writeLock().unlock();
>         }
>     }
> 
>     public T queryOne(Class<T> beanClass, String sql, Object... params) {
>         SqlPair key = new SqlPair(sql, params);
>         // 加读锁, 防止其它线程对缓存更改
>         lock.readLock().lock();
>         try {
>             T value = map.get(key);  // 从缓存里获取数据
>             if (value != null) {
>                 return value;
>             }
>         } finally {
>             lock.readLock().unlock();
>         }
> 
>         // 到这里说明缓存失效了
>         // 加写锁, 防止其它线程对缓存读取和更改
>         lock.writeLock().lock();
>         try {
>             // get 方法上面部分是可能多个线程进来的, 可能已经向缓存填充了数据
>             // 为防止重复查询数据库, 再次验证
>             T value = map.get(key);
>             if (value == null) {
>                 // 如果没有, 查询数据库
>                 value = genericDao.queryOne(beanClass, sql, params);
>                 map.put(key, value);  // 将查询的数据放入缓存
>             }
>             return value;
>         } finally {
>             lock.writeLock().unlock();
>         }
>     }
> 
>     // 作为 key 保证其是不可变的
>     class SqlPair {
>         private String sql;
>         private Object[] params;
>         public SqlPair(String sql, Object[] params) {
>             this.sql = sql;
>             this.params = params;
>         }
>         @Override
>         public boolean equals(Object o) {
>             if (this == o) {
>                 return true;
>             }
>             if (o == null || getClass() != o.getClass()) {
>                 return false;
>             }
>             SqlPair sqlPair = (SqlPair) o;
>             return sql.equals(sqlPair.sql) &&
>                 Arrays.equals(params, sqlPair.params);
>         }
>         @Override
>         public int hashCode() {
>             int result = Objects.hash(sql);
>             result = 31 * result + Arrays.hashCode(params);
>             return result;
>         }
>     }
> }
> ```
>
> > **注意**
> >
> > - 以上实现体现的是读写锁的应用，保证缓存和数据库的一致性，但有下面的问题没有考虑
> >   - 适合读多写少，如果写操作比较频繁，以上实现性能低
> >   - 没有考虑缓存容量
> >   - 没有考虑缓存过期
> >   - 只适合单机
> >   - 并发性还是低，目前只会用一把锁
> >   - 更新方法太过简单粗暴，清空了所有 key（考虑按类型分区或重新设计 key）
> > - 乐观锁实现：用 CAS 去更新 

##### 原理：读写锁原理

> 这部分的内容也暂时放放了，有待理解

#### 3.2 StampedLock 

该类自 JDK 8 加入，是为了进一步优化读性能，它的特点是在使用读锁、写锁时都必须配合 **戳** 使用 

加解读锁

```java
long stamp = lock.readLock();
lock.unlockRead(stamp);
```

加解写锁 

```java
long stamp = lock.writeLock();
lock.unlockWrite(stamp);
```

**乐观读**，StampedLock 支持 `tryOptimisticRead()` 方法（乐观读），读取完毕后需要做一次 **戳校验** 如果校验通过，表示这期间确实没有写操作，数据可以安全使用，如果校验没通过，需要重新获取读锁，保证数据安全。 

```java
long stamp = lock.tryOptimisticRead();
// 验戳
if(!lock.validate(stamp)){
    // 锁升级
}
```

提供一个 数据容器类 内部分别使用读锁保护数据的 `read()` 方法，写锁保护数据的 `write()` 方法 

```java
class DataContainerStamped{

    private int data;
    private final StampedLock lock = new StampedLock();

    public DataContainerStamped(int data) {
        this.data = data;
    }

    public int read(int readTime){
        // 获取乐观读锁
        long stamp = lock.tryOptimisticRead();
        LoggerUtils.LOGGER.debug("optimistic read locking...{}", stamp);
        Sleeper.sleep(readTime);
        if(lock.validate(stamp)){  // 对读锁进行校验
            LoggerUtils.LOGGER.debug("read finish...{}, data:{}", stamp, data);
            return data;
        }

        // 锁升级 - 读锁
        LoggerUtils.LOGGER.debug("updating to read lock... {}", stamp);
        try {
            stamp = lock.readLock();
            LoggerUtils.LOGGER.debug("read lock {}", stamp);
            Sleeper.sleep(readTime);
            LoggerUtils.LOGGER.debug("read finish...{}, data:{}", stamp, data);
            return data;
        }finally {
            LoggerUtils.LOGGER.debug("read unlock {}", stamp);
            lock.unlockRead(stamp);
        }
    }

    public void write(int newData){
        // 获取写锁去更新数据
        long stamp = lock.writeLock();
        LoggerUtils.LOGGER.debug("write lock {}", stamp);
        try {
            Sleeper.sleep(2);
            this.data = data;
        }finally {
            LoggerUtils.LOGGER.debug("write unlock {}", stamp);
            lock.unlockWrite(stamp);
        }
    }
}
```

测试 **读-读** 可以优化 

```java
DataContainerStamped dataContainer = new DataContainerStamped(1);

new Thread(()->{
    dataContainer.read(1);
}, "t1").start();
Sleeper.sleep(0.5);
new Thread(()->{
    dataContainer.read(0);
}, "t2").start();
```

输出结果，可以看到实际没有加读锁 

```java
16:25:04.007 cn.util.LoggerUtils [t1] - optimistic read locking...256
16:25:04.272 cn.util.LoggerUtils [t2] - optimistic read locking...256
16:25:04.272 cn.util.LoggerUtils [t2] - read finish...256, data:1
16:25:05.010 cn.util.LoggerUtils [t1] - read finish...256, data:1
```

测试 **读-写** 时优化读**补加读锁** 

```java
DataContainerStamped dataContainer = new DataContainerStamped(1);

new Thread(()->{
    dataContainer.read(1);
}, "t1").start();
Sleeper.sleep(0.5);
new Thread(()->{
    dataContainer.write(2);
}, "t2").start();
```

输出结果 

```
16:25:26.027 cn.util.LoggerUtils [t1] - optimistic read locking...256
16:25:26.295 cn.util.LoggerUtils [t2] - write lock 384
16:25:27.031 cn.util.LoggerUtils [t1] - updating to read lock... 256
16:25:28.296 cn.util.LoggerUtils [t2] - write unlock 384
16:25:28.298 cn.util.LoggerUtils [t1] - read lock 513
16:25:29.299 cn.util.LoggerUtils [t1] - read finish...513, data:1
16:25:29.299 cn.util.LoggerUtils [t1] - read unlock 513
```

> 注意
>
> - StampedLock 不支持条件变量
> - StampedLock 不支持可重入 

#### 4. Semaphore 

**基本使用**

[ˈsɛməˌfɔr] 信号量，用来限制能同时访问共享资源的线程上限。 

```java
// 1. 创建 semaphore 对象
Semaphore s = new Semaphore(3);  // 限制上限为3

// 2. 10个线程同时运行
for (int i = 0; i < 10; i++) {
    new Thread(()->{
        // 3. 获取许可
        try {
            s.acquire();  // 获得此信号量
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        try {
            LoggerUtils.LOGGER.debug("running...");
            Sleeper.sleep(1);
            LoggerUtils.LOGGER.debug("end...");
        }finally {
            // 4. 释放许可
            s.release();
        }
    }).start();
}
```

输出：

```java
16:43:46.717 cn.util.LoggerUtils [Thread-0] - running...
16:43:46.718 cn.util.LoggerUtils [Thread-1] - running...
16:43:46.718 cn.util.LoggerUtils [Thread-2] - running...
16:43:47.723 cn.util.LoggerUtils [Thread-0] - end...
16:43:47.723 cn.util.LoggerUtils [Thread-2] - end...
16:43:47.723 cn.util.LoggerUtils [Thread-1] - end...
16:43:47.723 cn.util.LoggerUtils [Thread-3] - running...
16:43:47.723 cn.util.LoggerUtils [Thread-4] - running...
16:43:47.723 cn.util.LoggerUtils [Thread-5] - running...
16:43:48.723 cn.util.LoggerUtils [Thread-3] - end...
16:43:48.723 cn.util.LoggerUtils [Thread-4] - end...
16:43:48.723 cn.util.LoggerUtils [Thread-5] - end...
16:43:48.723 cn.util.LoggerUtils [Thread-6] - running...
16:43:48.723 cn.util.LoggerUtils [Thread-7] - running...
16:43:48.723 cn.util.LoggerUtils [Thread-8] - running...
16:43:49.723 cn.util.LoggerUtils [Thread-6] - end...
16:43:49.723 cn.util.LoggerUtils [Thread-7] - end...
16:43:49.723 cn.util.LoggerUtils [Thread-9] - running...
16:43:49.724 cn.util.LoggerUtils [Thread-8] - end...
16:43:50.724 cn.util.LoggerUtils [Thread-9] - end...
```

##### 应用：Semaphore 应用

> 限制对共享资源的使用：semaphore 实现 
>
> - 使用 Semaphore 限流，在访问高峰期时，让请求线程阻塞，高峰期过去再释放许可，当然它只适合限制单机线程数量，并且仅是限制线程数，而不是限制资源数（例如连接数，请对比 Tomcat LimitLatch 的实现）
> - 用 Semaphore 实现简单连接池，对比『享元模式』下的实现（用wait notify），性能和可读性显然更好，注意下面的实现中线程数和数据库连接数是相等的 
>
> ```java
> class Pool{
>     // 1. 连接池大小
>     private final int poolSize;
>     // 2. 连接对象数组
>     private Connection[] connections;
>     // 3. 连接状态数组 0 表示空闲， 1 表示繁忙
>     private AtomicIntegerArray states;
> 
>     // new:创建 semaphore 对象
>     private Semaphore semaphore;
> 
>     // 4. 构造方法初始化
>     public Pool(int poolSize) {
>         this.poolSize = poolSize;
>         // 让许可数与资源数一致
>         this.semaphore = new Semaphore(poolSize);
>         this.connections = new Connection[poolSize];
>         this.states = new AtomicIntegerArray(new int[poolSize]);
>         for (int i = 0; i < poolSize; i++) {
>             connections[i] = new MockConnection("连接" + (i+1));
>         }
>     }
> 
>     // 5. 借连接
>     public Connection borrow(){
>         // 获取许可
>         try {
>             semaphore.acquire();
>         } catch (InterruptedException e) {
>             e.printStackTrace();
>         }
>         for (int i = 0; i < poolSize; i++) {
>             // 获取空闲连接
>             if(states.get(i) == 0){
>                 if(states.compareAndSet(i, 0, 1)){
>                     log.debug("borrow {}", connections[i]);
>                     return connections[i];
>                 }
>             }
>         }
>         // 不会执行到这里
>         return null;
>     }
> 
>     // 6. 归还连接
>     public void free(Connection connection){
>         for (int i = 0; i < poolSize; i++) {
>             if(connections[i] == connection){
>                 states.set(i, 0);
>                 log.debug("free {}", conn);
>                 // 有空闲了，则释放许可
>                 semaphore.release();
>                 break;
>             }
>         }
>     }
> }
> ```

##### 原理：Semaphore 原理

> 挖坑...

#### 5. CountdownLatch 

用来进行线程同步协作，等待所有线程完成倒计时。

其中构造参数用来初始化等待计数值，`await()` 用来等待计数归零，`countDown()` 用来让计数减一

```java
public static void main(String[] args) throws InterruptedException {

    CountDownLatch latch = new CountDownLatch(3);

    new Thread(() -> {
        LoggerUtils.LOGGER.debug("begin...");
        Sleeper.sleep(1);
        latch.countDown();
        LoggerUtils.LOGGER.debug("end...{}", latch.getCount());
    }).start();

    new Thread(() -> {
        LoggerUtils.LOGGER.debug("begin...");
        Sleeper.sleep(2);
        latch.countDown();
        LoggerUtils.LOGGER.debug("end...{}", latch.getCount());
    }).start();

    new Thread(() -> {
        LoggerUtils.LOGGER.debug("begin...");
        Sleeper.sleep(2.5);
        latch.countDown();
        LoggerUtils.LOGGER.debug("end...{}", latch.getCount());
    }).start();

    LoggerUtils.LOGGER.debug("main waiting...");
    latch.await();
    LoggerUtils.LOGGER.debug("wait end...");
}
```

输出：

```java
23:03:23.041 cn.util.LoggerUtils [main] - main waiting...
23:03:23.041 cn.util.LoggerUtils [Thread-2] - begin...
23:03:23.041 cn.util.LoggerUtils [Thread-0] - begin...
23:03:23.041 cn.util.LoggerUtils [Thread-1] - begin...
23:03:24.045 cn.util.LoggerUtils [Thread-0] - end...2
23:03:25.045 cn.util.LoggerUtils [Thread-1] - end...1
23:03:25.545 cn.util.LoggerUtils [Thread-2] - end...0
23:03:25.545 cn.util.LoggerUtils [main] - wait end...
```

可以配合线程池使用，改进如下：

```java
public static void main(String[] args) throws InterruptedException {

    CountDownLatch latch = new CountDownLatch(3);
    ExecutorService service = Executors.newFixedThreadPool(4);

    service.submit(() -> {
        LoggerUtils.LOGGER.debug("begin...");
        Sleeper.sleep(1);
        latch.countDown();
        LoggerUtils.LOGGER.debug("end...{}", latch.getCount());
    });

    service.submit(() -> {
        LoggerUtils.LOGGER.debug("begin...");
        Sleeper.sleep(2);
        latch.countDown();
        LoggerUtils.LOGGER.debug("end...{}", latch.getCount());
    });

    service.submit(() -> {
        LoggerUtils.LOGGER.debug("begin...");
        Sleeper.sleep(2.5);
        latch.countDown();
        LoggerUtils.LOGGER.debug("end...{}", latch.getCount());
    });

    service.submit(()->{
        try {
            LoggerUtils.LOGGER.debug("waiting...");
            latch.await();
            LoggerUtils.LOGGER.debug("wait end...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    service.shutdown();
}
```

输出：

```java
23:08:32.160 cn.util.LoggerUtils [pool-1-thread-4] - waiting...
23:08:32.160 cn.util.LoggerUtils [pool-1-thread-2] - begin...
23:08:32.160 cn.util.LoggerUtils [pool-1-thread-1] - begin...
23:08:32.160 cn.util.LoggerUtils [pool-1-thread-3] - begin...
23:08:33.163 cn.util.LoggerUtils [pool-1-thread-1] - end...2
23:08:34.163 cn.util.LoggerUtils [pool-1-thread-2] - end...1
23:08:34.663 cn.util.LoggerUtils [pool-1-thread-3] - end...0
23:08:34.663 cn.util.LoggerUtils [pool-1-thread-4] - wait end...
```

##### 应用：同步等待多线程准备完毕

> ```java
>public static void main(String[] args) throws InterruptedException {
> 
>  AtomicInteger num = new AtomicInteger(0);
>  ExecutorService service = Executors.newFixedThreadPool(10, (r)->{
>         return new Thread(r, "t"+num.getAndIncrement());
>     });
>     CountDownLatch latch = new CountDownLatch(10);
>     String[] all = new String[10];
>     Random r = new Random();
>    
>     for (int i = 0; i < 10; i++) {
>         int j = i;
>         service.submit(()->{
>             for (int k = 0; k <= 100; k++) {
>               try {
>                 Thread.sleep(r.nextInt(100));
>               }catch (InterruptedException e){
>               }
>               all[j] = Thread.currentThread().getName() + "(" + (k + "%") + ")";
>               System.out.print("\r" + Arrays.toString(all));  // "\r":在同一行刷新输出
>             }
>             latch.countDown();
>         });
>     }
>    
>     latch.await();
>     System.out.println("\n游戏开始...");
>   service.shutdown();
>    }
>    ```
>    
> 中间输出：
> 
> ```
>[t0(38%), t1(44%), t2(41%), t3(39%), t4(41%), t5(39%), t6(32%), t7(43%), t8(41%), t9(36%)]
> ```
>
> 最后输出：
> 
> ```
>[t0(100%), t1(100%), t2(100%), t3(100%), t4(100%), t5(100%), t6(100%), t7(100%), t8(100%), t9(100%)]
> 游戏开始...
>```

##### 应用：同步等待多个远程调用结束

> ```java
> @RestController
> public class TestCountDownlatchController {
>     @GetMapping("/order/{id}")
>     public Map<String, Object> order(@PathVariable int id) {
>         HashMap<String, Object> map = new HashMap<>();
>         map.put("id", id);
>         map.put("total", "2300.00");
>         sleep(2000);
>         return map;
>     }
>     @GetMapping("/product/{id}")
>     public Map<String, Object> product(@PathVariable int id) {
>         HashMap<String, Object> map = new HashMap<>();
>         if (id == 1) {
>             map.put("name", "小爱音箱");
>             map.put("price", 300);
>         } else if (id == 2) {
>             map.put("name", "小米手机");
>             map.put("price", 2000);
>         }
>         map.put("id", id);
>         sleep(1000);
>         return map;
>     }
>     @GetMapping("/logistics/{id}")
>     public Map<String, Object> logistics(@PathVariable int id) {
>         HashMap<String, Object> map = new HashMap<>();
>         map.put("id", id);
>         map.put("name", "中通快递");
>         sleep(2500);
>         return map;
>     }
>     private void sleep(int millis) {
>         try {
>             Thread.sleep(millis);
>         } catch (InterruptedException e) {
>             e.printStackTrace();
>         }
>     }
> }
> ```
>
> rest 远程调用 
>
> 1. 通过 CountDownLatch 方式同步：
>
>    ```java
>    RestTemplate restTemplate = new RestTemplate();
>    log.debug("begin");
>    ExecutorService service = Executors.newCachedThreadPool();
>    CountDownLatch latch = new CountDownLatch(4);
>    service.submit(() -> {
>        Map<String, Object> r =
>            restTemplate.getForObject("http://localhost:8080/order/{1}", Map.class, 1);
>        Log.debug("end order: {}", r);
>        latch.countDown();
>    });
>    service.submit(() -> {
>        Map<String, Object> r =
>            restTemplate.getForObject("http://localhost:8080/product/{1}", Map.class, 1);
>        Log.debug("end product: {}", r);
>        latch.countDown();
>    });
>    service.submit(() -> {
>        Map<String, Object> r =
>            restTemplate.getForObject("http://localhost:8080/product/{1}", Map.class, 2);
>        Log.debug("end prodect: {}", r);
>        latch.countDown();
>    });
>    service.submit(() -> {
>        Map<String, Object> r =
>            restTemplate.getForObject("http://localhost:8080/logistics/{1}", Map.class, 1);
>        Log.debug("end logistics: {}", r);
>        latch.countDown();
>    });
>    
>    latch.wait();
>    log.debug("执行完毕");
>    service.shutdown();
>    ```
>
>    问题：输出的内容都会在线程池的中，并没有传递给主线程，主线程如何获取线程执行结果？如下：
>
> 2. 通过 Future 
>
>    ```java
>    RestTemplate restTemplate = new RestTemplate();
>    log.debug("begin");
>    ExecutorService service = Executors.newCachedThreadPool();
>    Future<Map<String,Object>> f1 = service.submit(() -> {
>        Map<String, Object> r =
>            restTemplate.getForObject("http://localhost:8080/order/{1}", Map.class, 1);
>        return r;
>    });
>    Future<Map<String, Object>> f2 = service.submit(() -> {
>        Map<String, Object> r =
>            restTemplate.getForObject("http://localhost:8080/product/{1}", Map.class, 1);
>        return r;
>    });
>    Future<Map<String, Object>> f3 = service.submit(() -> {
>        Map<String, Object> r =
>            restTemplate.getForObject("http://localhost:8080/product/{1}", Map.class, 2);
>        return r;
>    });
>    Future<Map<String, Object>> f4 = service.submit(() -> {
>        Map<String, Object> r =
>            restTemplate.getForObject("http://localhost:8080/logistics/{1}", Map.class, 1);
>        return r;
>    });
>    System.out.println(f1.get());
>    System.out.println(f2.get());
>    System.out.println(f3.get());
>    System.out.println(f4.get());
>    log.debug("执行完毕");
>    service.shutdown();
>    ```
>
>    执行结果 
>
>    ```java
>    19:51:39.711 c.TestCountDownLatch [main] - begin
>    {total=2300.00, id=1}
>    {price=300, name=小爱音箱, id=1}
>    {price=2000, name=小米手机, id=2}
>    {name=中通快递, id=1}
>    19:51:42.407 c.TestCountDownLatch [main] - 执行完毕
>    ```

#### 6. CyclicBarrier 

[ˈsaɪklɪk ˈbæriɚ] 循环栅栏，用来进行线程协作，等待线程满足某个计数。构造时设置『计数个数』，每个线程执行到某个需要“同步”的时刻调用 `await()` 方法进行等待，当等待的线程数满足『计数个数』时，继续执行 

```java
CyclicBarrier cb = new CyclicBarrier(2);  // 个数为2时才会继续执行

new Thread(()->{
    System.out.println("线程1开始.."+new Date());
    try {
        cb.await(); // 当个数不足时，等待
    } catch (InterruptedException | BrokenBarrierException e) {
        e.printStackTrace();
    }
    System.out.println("线程1继续向下运行..."+new Date());
}).start();

new Thread(()->{
    System.out.println("线程2开始.."+new Date());
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) { }
    try {
        cb.await(); // 2 秒后，线程个数够2，继续运行
    } catch (InterruptedException | BrokenBarrierException e) {
        e.printStackTrace();
    }
    System.out.println("线程2继续向下运行..."+new Date());
}).start();
```

> 注意 CyclicBarrier 与 CountDownLatch 的主要区别在于 CyclicBarrier 是可以重用的 
>
> CyclicBarrier 可以被比喻为**『人满发车』** 

#### 7. 线程安全集合类概述 

![线程安全集合类](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251549434.PNG)

线程安全集合类可以分为三大类： 

- 遗留的线程安全集合如 `Hashtable` ， `Vector`
- 使用 `Collections` 装饰的线程安全集合，如：
  - `Collections.synchronizedCollection`
  - `Collections.synchronizedList`
  - `Collections.synchronizedMap`
  - `Collections.synchronizedSet`
  - `Collections.synchronizedNavigableMap`
  - `Collections.synchronizedNavigableSet`
  - `Collections.synchronizedSortedMap`
  - `Collections.synchronizedSortedSet`
- `java.util.concurrent.*` 

重点介绍 `java.util.concurrent.*` 下的线程安全集合类，可以发现它们有规律，里面包含三类关键词：`Blocking、CopyOnWrite、Concurrent`

- Blocking 大部分实现基于锁，并提供用来阻塞的方法
- CopyOnWrite 之类容器修改开销相对较重
- Concurrent 类型的容器
  - 内部很多操作**使用 cas 优化**，一般可以提供较高吞吐量
  - 弱一致性
    - 遍历时弱一致性，例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续进行遍历，这时内容是旧的
    - 求大小弱一致性，size 操作未必是 100% 准确
    - 读取弱一致性 

> 遍历时如果发生了修改，对于非安全容器来讲，使用 fail-fast 机制也就是让遍历立刻失败，抛出ConcurrentModificationException，不再继续遍历 

#### 8. ConcurrentHashMap

> 可见之前的博文：[Java：ConcurrentHashMap类小记](https://www.cnblogs.com/zhuchengchao/p/14446548.html)

**练习：单词计数** 

生成测试数据

```java
static final String ALPHA = "abcedfghijklmnopqrstuvwxyz";

public static void main(String[] args) {

    int length = ALPHA.length();
    int count = 200;
    List<String> list = new ArrayList<>(length * count);
    for (int i = 0; i < length; i++) {
        char ch = ALPHA.charAt(i);
        for (int j = 0; j < count; j++) {
            list.add(String.valueOf(ch));
        }
    }

    Collections.shuffle(list);
    for (int i = 0; i < 26; i++) {
        try (PrintWriter out = new PrintWriter(
            new OutputStreamWriter(
                new FileOutputStream("./tmp/"+(i+1)+".txt")))){
            String collect = list.subList(i*count, (i+1)*count).stream().collect(Collectors.joining("\n"));
            out.print(collect);
        }catch (IOException e){
            e.printStackTrace();
        }
        finally { }
    }
}
```

模版代码，模版代码中封装了多线程读取文件的代码

```java
private static <V> void demo(Supplier<Map<String, V>> supplier,
                             BiConsumer<Map<String, V>, List<String>> consumer){
    Map<String, V> counterMap = supplier.get();
    List<Thread> ts = new ArrayList<>();

    for (int i = 1; i <= 26; i++) {
        int idx = i;
        Thread thread = new Thread(()->{
            List<String> words = readFromFile(idx);
            consumer.accept(counterMap, words);
        });
        ts.add(thread);
    }

    ts.forEach(t->t.start());
    ts.forEach(t->{
        try {
            t.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });

    System.out.println(counterMap);
}

public static List<String> readFromFile(int i){
    ArrayList<String> words = new ArrayList<>();
    try (
        BufferedReader in = new BufferedReader(
            new InputStreamReader(
                new FileInputStream("./tmp/"+i+".txt")))) {
        while (true){
            String word = in.readLine();
            if(word == null){
                break;
            }
            words.add(word);
        }
    }catch (IOException e){
        e.printStackTrace();
    }
    return words;
}
```

你要做的是实现两个参数

- 一是提供一个 map 集合，用来存放每个单词的计数结果，key 为单词，value 为计数
- 二是提供一组操作，保证计数的安全性，会传递 map 集合以及 单词 List 

正确结果输出应该是每个单词出现 200 次 

```java
{a=200, b=200, c=200, d=200, e=200, f=200, g=200, h=200, i=200, j=200, k=200, l=200, m=200, n=200, o=200, p=200, q=200, r=200, s=200, t=200, u=200, v=200, w=200, x=200, y=200, z=200}
```

下面的实现为： 

```java
demo(
    // 创建 map 集合
    ()->new HashMap<String, Integer>(),
    // 进行计数
    (map, words)->{
        for(String word: words){
            Integer count = map.get(word);
            int newValue = count==null ? 1 : count+1;
            map.put(word, newValue);
        }
    }
);
```

有没有问题？请改进 

参考解答1：

```java
demo(
    ()->new ConcurrentHashMap<String, LongAdder>(),
    (map, words)->{
        for(String word: words){
            // 如果缺少一个key，则计算生成一个值，然后将key value放入map中
            LongAdder value = map.computeIfAbsent(word, (key)->new LongAdder());
            // 执行累加
            value.increment();
        }
    }
);
```

参考解答2：

```java
demo(
    ()->new ConcurrentHashMap<String, Integer>(),
    (map, words)->{
        for(String word: words){
            // 函数式编程，无需原子变量
            map.merge(word, 1, Integer::sum);
        }
    }
);
```

##### 原理：ConcurrentHashMap 原理

> 挖坑:dog2:

> 后续还有些关于 BlockingQueue、LinkedBlockingQueue、ConcurrentLinkedQueue等内容，同样对其也缺乏一定的理解，等有一定功力后再回来填坑了

#### 9. CopyOnWriteArrayList 

CopyOnWriteArraySet 是它的马甲底层实现采用了 **写入时拷贝的思想**，增删改操作会将底层数组拷贝一份，更改操作在新数组上执行，这时不影响其它线程的**并发读**，**读写分离**。 

以新增为例： 

```java
public boolean add(E e) {
    synchronized (lock) {
        // 获取旧的数组
        Object[] es = getArray();
        int len = es.length;
        // 拷贝新的数组（这里是比较耗时的操作，但不影响其它读线程）
        es = Arrays.copyOf(es, len + 1);
        // 添加新元素
        es[len] = e;
        // 替换旧的数组
        setArray(es);
        return true;
    }
}
```

> 这里的源码版本是 Java 11，在 Java 1.8 中使用的是可重入锁而不是 synchronized，如下
>
> ```java
> public boolean add(E e) {
>     // 用了可重入锁
>     final ReentrantLock lock = this.lock;
>     lock.lock();
>     try {
>         // 获取旧的数组
>         Object[] elements = getArray();
>         int len = elements.length;
>         // 拷贝新的数组（这里是比较耗时的操作，但不影响其它读线程）
>         Object[] newElements = Arrays.copyOf(elements, len + 1);
>         // 添加新元素
>         newElements[len] = e;
>         // 替换旧的数组
>         setArray(newElements);
>         return true;
>     } finally {
>         lock.unlock();
>     }
> }
> ```

其它读操作并未加锁，例如： 

```java
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    for (Object x : getArray()) {
        @SuppressWarnings("unchecked") E e = (E) x;
        action.accept(e);
    }
}
```

适合『**读多写少**』的应用场景 

**get 弱一致性体现**

![get 弱一致性](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251550703.PNG)

| 时间点 | 操作                         |
| ------ | ---------------------------- |
| 1      | Thread-0 getArray()          |
| 2      | Thread-1 getArray()          |
| 3      | Thread-1 setArray(arrayCopy) |
| 4      | Thread-0 array[index]        |

**代码体现：迭代器弱一致性**

```java
CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>();
list.add(1);
list.add(2);
list.add(3);
Iterator<Integer> iter = list.iterator();
new Thread(() -> {
    list.remove(0);
    System.out.println(list);  // 打印 2 3
}).start();
TimeUnit.SECONDS.sleep(1l);
while (iter.hasNext()) {
    System.out.println(iter.next());  // 打印 1 2 3
}
```

> 不要觉得弱一致性就不好
>
> - 数据库的 MVCC（Multi-Version Concurrency Control，多版本并发控制） 都是弱一致性的表现
> - 并发高和一致性是矛盾的，需要权衡 

