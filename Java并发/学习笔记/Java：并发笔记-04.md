# Java：并发笔记-04

> 说明：观看了 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444/)** 的课程 [java并发编程](https://www.bilibili.com/video/BV16J411h7Rd) 后所作的学习笔记，总计拆分为9篇笔记，此为4/9。

### 本章内容-3

- 线程状态转换
- 活跃性
- Lock

### 3.10 重新理解线程状态转换

![线程的6种状态](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547457.PNG)

假设有线程 `Thread t`

#### 情况 1 `NEW --> RUNNABLE`

当调用 `t.start()` 方法时，由 `NEW --> RUNNABLE`

#### 情况 2 `RUNNABLE <--> WAITING`

t 线程用 `synchronized(obj)` 获取了对象锁后

- 调用 `obj.wait()` 方法时，t 线程从 `RUNNABLE --> WAITING`
- 调用 `obj.notify()` ， `obj.notifyAll()` ， `t.interrupt()` 时
  - 竞争锁成功，t 线程从 `WAITING --> RUNNABLE`
  - 竞争锁失败，t 线程从 `WAITING --> BLOCKED`

```java
final static Object obj = new Object();

public static void main(String[] args) {

    new Thread(() -> {
        synchronized (obj) {
            LoggerUtils.LOGGER.debug("执行....");
            try {
                obj.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LoggerUtils.LOGGER.debug("其它代码...."); // 加上线程断点
        }
    },"t1").start();

    new Thread(() -> {
        synchronized (obj) {
            LoggerUtils.LOGGER.debug("执行....");
            try {
                obj.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LoggerUtils.LOGGER.debug("其它代码...."); // 加上线程断点
        }
    },"t2").start();

    Sleeper.sleep(0.5);
    LoggerUtils.LOGGER.debug("唤醒 obj 上其它线程");
    synchronized (obj) {
        obj.notifyAll(); // 唤醒obj上所有等待线程 断点
    }
}
```

#### 情况 3 `RUNNABLE <--> WAITING`

- 当前线程调用 `t.join()` 方法时，当前线程从 `RUNNABLE --> WAITING`
  - 注意是当前线程在 t 线程对象的监视器上等待，即：调用join的线程等待t线程执行完成
- t 线程运行结束，或调用了当前线程的 interrupt() 时，当前线程从 `WAITING --> RUNNABLE`

#### 情况 4 `RUNNABLE <--> WAITING`

- 当前线程调用 `LockSupport.park()` 方法会让当前线程从 `RUNNABLE --> WAITING`
- 调用 `LockSupport.unpark(目标线程)` 或调用了线程的 `interrupt()` ，会让目标线程从 `WAITING -->RUNNABLE`

#### 情况 5 `RUNNABLE <--> TIMED_WAITING`

t 线程用 `synchronized(obj)` 获取了对象锁后

- 调用 `obj.wait(long n)` 方法时，t 线程从 `RUNNABLE --> TIMED_WAITING`
- t 线程等待时间超过了 n 毫秒，或调用 `obj.notify()` ， `obj.notifyAll()` ， `t.interrupt()` 时
  - 竞争锁成功，t 线程从 `TIMED_WAITING --> RUNNABLE`
  - 竞争锁失败，t 线程从 `TIMED_WAITING --> BLOCKED`

#### 情况 6 `RUNNABLE <--> TIMED_WAITING`

- 当前线程调用 `t.join(long n)` 方法时，当前线程从 `RUNNABLE --> TIMED_WAITING`
  - 注意是当前线程在t 线程对象的监视器上等待，即：调用join的线程等待t线程执行完成
- 当前线程等待时间超过了 n 毫秒，或 t 线程运行结束，或调用了当前线程的 `interrupt()` 时，当前线程从`TIMED_WAITING --> RUNNABLE`

#### 情况 7 `RUNNABLE <--> TIMED_WAITING`

- 当前线程调用 `Thread.sleep(long n)` ，当前线程从 `RUNNABLE --> TIMED_WAITING`
- 当前线程等待时间超过了 n 毫秒，当前线程从 `TIMED_WAITING --> RUNNABLE`

#### 情况 8 `RUNNABLE <--> TIMED_WAITING`

- 当前线程调用 `LockSupport.parkNanos(long nanos)` 或 `LockSupport.parkUntil(long millis)` 时，当前线程从 `RUNNABLE --> TIMED_WAITING`
- 调用 `LockSupport.unpark(目标线程)` 或调用了线程的 `interrupt() `，或是等待超时，会让目标线程从`TIMED_WAITING--> RUNNABLE`

#### 情况 9 `RUNNABLE <--> BLOCKED`

- t 线程用 `synchronized(obj)` 获取了对象锁时如果竞争失败，从 `RUNNABLE --> BLOCKED`
- 持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 `BLOCKED` 的线程重新竞争，如果其中 t 线程竞争成功，从 `BLOCKED --> RUNNABLE` ，其它失败的线程仍然 `BLOCKED`

#### 情况 10 `RUNNABLE <--> TERMINATED`

当前线程所有代码运行完毕，进入 `TERMINATED`

### 3.11 多把锁

**多把不相干的锁**

一间大屋子有两个功能：睡觉、学习，互不相干。

现在小南要学习，小女要睡觉，但如果只用一间屋子（一个对象锁）的话，那么并发度很低。

解决方法是准备多个房间（多个对象锁）

例如：

```java
public class BigRoom {

    public void sleep(){
        synchronized (this){
            LoggerUtils.LOGGER.debug("sleeping 2 hours...");
            Sleeper.sleep(2);
        }
    }

    public void study(){
        synchronized (this){
            LoggerUtils.LOGGER.debug("study 1 hours...");
            Sleeper.sleep(1);
        }
    }
}
```

执行：

```java
public static void main(String[] args) {
    BigRoom bigRoom = new BigRoom();

    LoggerUtils.LOGGER.debug("start...");

    new Thread(()->{
        bigRoom.sleep();
    }, "小南").start();

    new Thread(()->{
        bigRoom.study();
    }, "小女").start();
}
```

某次结果：

```java
21:17:15.862 cn.util.LoggerUtils [main] - start...
21:17:15.900 cn.util.LoggerUtils [小南] - sleeping 2 hours...
21:17:17.902 cn.util.LoggerUtils [小女] - study 1 hours...
```

改进：

```java
private final Object studyRoom = new Object();
private final Object bedRoom = new Object();

public void sleep(){
    synchronized (bedRoom){
        LoggerUtils.LOGGER.debug("sleeping 2 hours...");
        Sleeper.sleep(2);
    }
}

public void study(){
    synchronized (studyRoom){
        LoggerUtils.LOGGER.debug("study 1 hours...");
        Sleeper.sleep(1);
    }
}
```

某次结果：

```java
21:18:54.105 cn.util.LoggerUtils [main] - start...
21:18:54.147 cn.util.LoggerUtils [小南] - sleeping 2 hours...
21:18:54.148 cn.util.LoggerUtils [小女] - study 1 hours...
```

将锁的粒度细分

- 好处，是可以增强并发度
- 坏处，如果一个线程需要同时获得多把锁，就容易发生死锁

### 3.12 活跃性

#### 死锁

有这样的情况：一个线程需要同时获取多把锁，这时就容易发生死锁

t1 线程 获得 A对象 锁，接下来想获取 B对象 的锁 

t2 线程 获得 B对象 锁，接下来想获取 A对象 的锁 

```java
public static void main(String[] args) {
    Object A = new Object();
    Object B = new Object();

    Thread t1 = new Thread(() -> {
        synchronized (A) {
            LoggerUtils.LOGGER.debug("lock A");
            Sleeper.sleep(1);
            synchronized (B) {
                LoggerUtils.LOGGER.debug("lock B");
                LoggerUtils.LOGGER.debug("操作...");
            }
        }
    }, "t1");

    Thread t2 = new Thread(() -> {
        synchronized (B) {
            LoggerUtils.LOGGER.debug("lock B");
            Sleeper.sleep(1);
            synchronized (A) {
                LoggerUtils.LOGGER.debug("lock A");
                LoggerUtils.LOGGER.debug("操作...");
            }
        }
    }, "t2");

    t1.start();
    t2.start();
}
```

结果：

```java
21:21:10.342 cn.util.LoggerUtils [t1] - lock A
21:21:10.342 cn.util.LoggerUtils [t3] - lock B
// 无限等待
```

#### 定位死锁

检测死锁可以使用 jconsole工具，或者使用 jps 定位进程 id，再用 jstack 定位死锁：

* jps 定位进程id

```java
C:\Users\ZhuCC>jps
74480 Jps
10184 Test10  // JVM进程
28572 Launcher
48460
```

* 通过 jstack 定位死锁

```bash
C:\Users\ZhuCC>jstack 10184
2020-08-28 21:24:53
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.241-b07 mixed mode):

"DestroyJavaVM" #13 prio=5 os_prio=0 tid=0x00000000033e3000 nid=0x16f4 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   
"t2" #12 prio=5 os_prio=0 tid=0x000000001ea64800 nid=0xf4f4 waiting for monitor entry [0x000000001f45f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at cn.xyc.n4.Test10.lambda$main$1(Test10.java:28)
        - waiting to lock <0x000000076b3fc6b0> (a java.lang.Object)
        - locked <0x000000076b3fc6c0> (a java.lang.Object)
        at cn.xyc.n4.Test10$$Lambda$2/1480010240.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

"t1" #11 prio=5 os_prio=0 tid=0x000000001e24e800 nid=0xfe68 waiting for monitor entry [0x000000001f35e000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at cn.xyc.n4.Test10.lambda$main$0(Test10.java:17)
        - waiting to lock <0x000000076b3fc6c0> (a java.lang.Object)
        - locked <0x000000076b3fc6b0> (a java.lang.Object)
        at cn.xyc.n4.Test10$$Lambda$1/999966131.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
        
Found one Java-level deadlock:
=============================
"t2":
  waiting to lock monitor 0x00000000034d8da8 (object 0x000000076b3fc6b0, a java.lang.Object),
  which is held by "t1"
"t1":
  waiting to lock monitor 0x00000000034dac98 (object 0x000000076b3fc6c0, a java.lang.Object),
  which is held by "t2"

Java stack information for the threads listed above:
===================================================
"t2":
        at cn.xyc.n4.Test10.lambda$main$1(Test10.java:28)
        - waiting to lock <0x000000076b3fc6b0> (a java.lang.Object)
        - locked <0x000000076b3fc6c0> (a java.lang.Object)
        at cn.xyc.n4.Test10$$Lambda$2/1480010240.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"t1":
        at cn.xyc.n4.Test10.lambda$main$0(Test10.java:17)
        - waiting to lock <0x000000076b3fc6c0> (a java.lang.Object)
        - locked <0x000000076b3fc6b0> (a java.lang.Object)
        at cn.xyc.n4.Test10$$Lambda$1/999966131.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

- 避免死锁要注意加锁顺序
- 另外如果由于某个线程进入了死循环，导致其它线程一直等待，对于这种情况 linux 下可以通过 top 先定位到CPU 占用高的 Java 进程，再利用 top -Hp 进程 id 来定位是哪个线程，最后再用 jstack 排查

#### 哲学家就餐问题

![哲学家就餐](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547801.PNG)

有五位哲学家，围坐在圆桌旁。

- 他们只做两件事，思考和吃饭，思考一会吃口饭，吃完饭后接着思考。
- 吃饭时要用两根筷子吃，桌上共有 5 根筷子，每位哲学家左右手边各有一根筷子。
- 如果筷子被身边的人拿着，自己就得等待

筷子类：

```java
class Chopstick{
    String name;

    public Chopstick(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Chopstick{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

哲学家类：

```java
public class Philosopher extends Thread {

    Chopstick left;
    Chopstick right;

    public Philosopher(String name, Chopstick left, Chopstick right){
        super(name);
        this.left = left;
        this.right = right;
    }

    private void eat(){
        LoggerUtils.LOGGER.debug("eating...");
        Sleeper.sleep(1);
    }

    @Override
    public void run() {
        while (true){
            // 获得左手筷子
            synchronized (left){
                // 获得右手筷子
                synchronized (right){
                    // 吃饭
                    eat();
                }
                // 放下右手筷子
            }
            // 放下左手筷子
        }
    }
}

```

就餐：

```java
public static void main(String[] args) {

    Chopstick c1 = new Chopstick("1");
    Chopstick c2 = new Chopstick("2");
    Chopstick c3 = new Chopstick("3");
    Chopstick c4 = new Chopstick("4");
    Chopstick c5 = new Chopstick("5");
    new Philosopher("苏格拉底", c1, c2).start();
    new Philosopher("柏拉图", c2, c3).start();
    new Philosopher("亚里士多德", c3, c4).start();
    new Philosopher("赫拉克利特", c4, c5).start();
    new Philosopher("阿基米德", c5, c1).start();
}
```

执行不多会儿，就执行不下去了：

```
21:40:29.880 cn.util.LoggerUtils [亚里士多德] - eating...
21:40:29.880 cn.util.LoggerUtils [苏格拉底] - eating...
21:40:30.883 cn.util.LoggerUtils [亚里士多德] - eating...
21:40:30.883 cn.util.LoggerUtils [阿基米德] - eating...
21:40:31.884 cn.util.LoggerUtils [亚里士多德] - eating...
21:40:32.884 cn.util.LoggerUtils [亚里士多德] - eating...
21:40:33.884 cn.util.LoggerUtils [亚里士多德] - eating...
21:40:34.885 cn.util.LoggerUtils [柏拉图] - eating...
```

使用 jconsole 检测死锁，发现：

```java
-------------------------------------------------------------------------
名称: 阿基米德
状态: cn.itcast.Chopstick@1540e19d (筷子1) 上的BLOCKED, 拥有者: 苏格拉底
总阻止数: 2, 总等待数: 1
    
堆栈跟踪:
cn.itcast.Philosopher.run(TestDinner.java:48)
	- 已锁定 cn.itcast.Chopstick@6d6f6e28 (筷子5)
-------------------------------------------------------------------------
名称: 苏格拉底
状态: cn.itcast.Chopstick@677327b6 (筷子2) 上的BLOCKED, 拥有者: 柏拉图
总阻止数: 2, 总等待数: 1

堆栈跟踪:
cn.itcast.Philosopher.run(TestDinner.java:48)
	- 已锁定 cn.itcast.Chopstick@1540e19d (筷子1)
-------------------------------------------------------------------------
名称: 柏拉图
状态: cn.itcast.Chopstick@14ae5a5 (筷子3) 上的BLOCKED, 拥有者: 亚里士多德
总阻止数: 2, 总等待数: 0
    
堆栈跟踪:
cn.itcast.Philosopher.run(TestDinner.java:48)
	- 已锁定 cn.itcast.Chopstick@677327b6 (筷子2)
-------------------------------------------------------------------------
名称: 亚里士多德
状态: cn.itcast.Chopstick@7f31245a (筷子4) 上的BLOCKED, 拥有者: 赫拉克利特
总阻止数: 1, 总等待数: 1
    
堆栈跟踪:
cn.itcast.Philosopher.run(TestDinner.java:48)
	- 已锁定 cn.itcast.Chopstick@14ae5a5 (筷子3)
-------------------------------------------------------------------------
名称: 赫拉克利特
状态: cn.itcast.Chopstick@6d6f6e28 (筷子5) 上的BLOCKED, 拥有者: 阿基米德
总阻止数: 2, 总等待数: 0
    
堆栈跟踪:
cn.itcast.Philosopher.run(TestDinner.java:48)
	- 已锁定 cn.itcast.Chopstick@7f31245a (筷子4)

```

这种线程没有按预期结束，执行不下去的情况，归类为【活跃性】问题，除了死锁以外，还有活锁和饥饿者两种情况

#### 活锁

活锁出现在两个线程互相改变对方的结束条件，最后谁也无法结束，例如：

```java
public class TestLiveLock {

    static volatile int count = 10;
    static final Object lock = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            // 期望减到 0 退出循环
            while (count > 0) {
                Sleeper.sleep(0.2);
                count--;
                LoggerUtils.LOGGER.debug("count: {}", count);
            }
        }, "t1").start();
        
        new Thread(() -> {
            // 期望超过 20 退出循环
            while (count < 20) {
                Sleeper.sleep(0.2);
                count++;
                LoggerUtils.LOGGER.debug("count: {}", count);
            }
        }, "t2").start();
    }
}
```

#### 饥饿

很多教程中把饥饿定义为，一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束，饥饿的情况不易演示，讲读写锁时会涉及饥饿问题

下面我讲一下我遇到的一个线程饥饿的例子，先来看看使用顺序加锁的方式解决之前的死锁问题

![饥饿](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547677.PNG)

顺序加锁的解决方案：

![顺序加锁的解决方案](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547859.PNG)

```java
new Philosopher("苏格拉底", c1, c2).start();
new Philosopher("柏拉图", c2, c3).start();
new Philosopher("亚里士多德", c3, c4).start();
new Philosopher("赫拉克利特", c4, c5).start();
new Philosopher("阿基米德", c1, c5).start();  // 这里修改了，先拿到c1再去拿c5
```

### 3.13 ReentrantLock

相对于 synchronized 它具备如下特点

- **可中断**
- **可以设置超时时间**
- **可以设置为公平锁**
- **支持多个条件变量**

与 synchronized 一样，都支持可重入

基本语法

```java
// 获取锁——> reentrantLock.lock(); 放在try里面还是外面都一样，根据自己的习惯
reentrantLock.lock();
try {
	// 临界区
} finally {
	// 释放锁
	reentrantLock.unlock();
}
```

#### 可重入

可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁（synchronize，reentrantLock）

如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住

```java
static ReentrantLock lock = new ReentrantLock();

public static void main(String[] args) {
    method1();
}

public static void method1(){
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("execute method1");
        method2();
    }finally {
        lock.unlock();
    }
}

public static void method2(){
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("execute method2");
        method3();
    }finally {
        lock.unlock();
    }
}

public static void method3(){
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("execute method2");
    }finally {
        lock.unlock();
    }
}
```

输出：

```
21:49:10.299 cn.util.LoggerUtils [main] - execute method1
21:49:10.300 cn.util.LoggerUtils [main] - execute method2
21:49:10.301 cn.util.LoggerUtils [main] - execute method2
```

#### 可打断

示例：

```java
ReentrantLock lock = new ReentrantLock();

Thread t1 = new Thread(()->{
    LoggerUtils.LOGGER.debug("线程1->启动...");
    try {
        lock.lockInterruptibly();  // 可打断的上锁
    } catch (InterruptedException e) {
        e.printStackTrace();
        LoggerUtils.LOGGER.debug("等锁的过程中被打断");
        return;
    }

    try {
        LoggerUtils.LOGGER.debug("线程1->获得了锁");
    }finally {
        lock.unlock();
    }
}, "t1");

lock.lock();
LoggerUtils.LOGGER.debug("主线程->获得了锁");
t1.start();
try {
    Sleeper.sleep(1);
    t1.interrupt();
    LoggerUtils.LOGGER.debug("线程1->执行打断");
}finally {
    lock.unlock();
}
```

输出：

```java
21:55:55.537 cn.util.LoggerUtils [main] - 主线程->获得了锁
21:55:55.539 cn.util.LoggerUtils [t1] - 线程1->启动...
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireInterruptibly(AbstractQueuedSynchronizer.java:898)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:1222)
	at java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:335)
	at cn.xyc.n4.Test12.lambda$main$0(Test12.java:16)
	at java.lang.Thread.run(Thread.java:748)
21:55:56.541 cn.util.LoggerUtils [main] - 线程1->执行打断
21:55:56.542 cn.util.LoggerUtils [t1] - 等锁的过程中被打断
```

注意如果是不可中断模式，那么即使使用了 interrupt 也不会让等待中断：

```java
ReentrantLock lock = new ReentrantLock();

Thread t1 = new Thread(()->{
    LoggerUtils.LOGGER.debug("线程1->启动...");
    lock.lock();  // 不可中断模式上锁
    try {
        LoggerUtils.LOGGER.debug("线程1->获得了锁");
    }finally {
        lock.unlock();
    }
}, "t1");

lock.lock();
LoggerUtils.LOGGER.debug("主线程->获得了锁");
t1.start();
try {
    Sleeper.sleep(1);
    t1.interrupt();
    LoggerUtils.LOGGER.debug("线程1->执行打断");
}finally {
    LoggerUtils.LOGGER.debug("主线程->释放锁");
    lock.unlock();
}
```

输出：

```java
21:57:49.530 cn.util.LoggerUtils [main] - 主线程->获得了锁
21:57:49.534 cn.util.LoggerUtils [t1] - 线程1->启动...
21:57:50.535 cn.util.LoggerUtils [main] - 线程1->执行打断
21:57:50.535 cn.util.LoggerUtils [main] - 主线程->释放锁
21:57:50.535 cn.util.LoggerUtils [t1] - 线程1->获得了锁  // 即线程1没有被中断
```

#### 锁超时

立刻失败

```java
public static void main(String[] args) {
    ReentrantLock lock = new ReentrantLock();

    Thread t1 = new Thread(() -> {
        LoggerUtils.LOGGER.debug("线程1->启动...");
        if (!lock.tryLock()) {
            LoggerUtils.LOGGER.debug("线程1->获取锁立刻失败，返回");
            return;
        }
        try {
            LoggerUtils.LOGGER.debug("线程1->获得了锁");
        } finally {
            lock.unlock();
        }
    }, "t1");

    lock.lock();
    LoggerUtils.LOGGER.debug("主线程->获得了锁");
    t1.start();
    try {
        Sleeper.sleep(2);
    }finally {
        LoggerUtils.LOGGER.debug("主线程->释放锁");
        lock.unlock();
    }
}
```

输出：

```java
22:06:06.471 cn.util.LoggerUtils [main] - 主线程->获得了锁
22:06:06.475 cn.util.LoggerUtils [t1] - 线程1->启动...
22:06:06.475 cn.util.LoggerUtils [t1] - 线程1->获取锁立刻失败，返回
22:06:08.477 cn.util.LoggerUtils [main] - 主线程->释放锁
```

超时失败：

```java
public static void main(String[] args) {
    ReentrantLock lock = new ReentrantLock();

    Thread t1 = new Thread(() -> {
        LoggerUtils.LOGGER.debug("线程1->启动...");
        try {
            if (!lock.tryLock(1, TimeUnit.SECONDS)) {
                LoggerUtils.LOGGER.debug("线程1->获取等待 1s 后失败，返回");
                return;
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        try {
            LoggerUtils.LOGGER.debug("线程1->获得了锁");
        } finally {
            lock.unlock();
        }
    }, "t1");

    lock.lock();
    LoggerUtils.LOGGER.debug("主线程->获得了锁");
    t1.start();
    try {
        Sleeper.sleep(2);
    }finally {
        LoggerUtils.LOGGER.debug("主线程->释放锁");
        lock.unlock();
    }
}
```

输出：

```java
22:09:39.667 cn.util.LoggerUtils [main] - 主线程->获得了锁
22:09:39.669 cn.util.LoggerUtils [t1] - 线程1->启动...
22:09:40.672 cn.util.LoggerUtils [t1] - 线程1->获取等待 1s 后失败，返回
22:09:41.671 cn.util.LoggerUtils [main] - 主线程->释放锁
```

使用 tryLock 解决哲学家就餐问题：

```java
// 筷子类修改为
class Chopstick extends ReentrantLock{
    String name;

    public Chopstick(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Chopstick{" +
                "name='" + name + '\'' +
                '}';
    }
}

// 哲学家类修改为：
public class Philosopher extends Thread {

    Chopstick left;
    Chopstick right;

    public Philosopher(String name, Chopstick left, Chopstick right){
        super(name);
        this.left = left;
        this.right = right;
    }

    private void eat(){
        LoggerUtils.LOGGER.debug("eating...");
        Sleeper.sleep(1);
    }

    @Override
    public void run() {
        while (true){
            // 获得左手筷子
            if(left.tryLock()){
                try {
                    // 获得右手筷子
                    if(right.tryLock()){
                        try {
                            // 吃饭
                            eat();
                        }finally {
                            // 放下右手筷子
                            right.unlock();
                        }
                    }
                }finally {
                    // 放下左手筷子
                    left.unlock();
                }
            }
        }
    }
}
```

#### 公平锁

ReentrantLock 默认是不公平的：

> 可选择相应的构造函数进行设置，是否公平：
>
> ```java
> public ReentrantLock(boolean fair) {
> 	sync = fair ? new FairSync() : new NonfairSync();
> }
> ```

```java
ReentrantLock lock = new ReentrantLock(false);  // 不公平
// 主线程上锁
lock.lock();
// 创建线程，但是现在主线程拿着锁，故都进入等待状态
for (int i = 0; i < 500; i++) {
    new Thread(() -> {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " running...");
        } finally {
            lock.unlock();
        }
    }, "t" + i).start();
}

try {
    Thread.sleep(1000);
}finally {
    System.out.println("1s后主线程释放所");
    lock.unlock();
}

// 1s 之后去争抢锁
new Thread(() -> {
    System.out.println(Thread.currentThread().getName() + " start...");
    lock.lock();
    try {
        System.out.println(Thread.currentThread().getName() + " running...");
    } finally {
        lock.unlock();
    }
}, "强行插入").start();
```

强行插入，有机会在中间输出

> **注意**：该实验不一定总能复现 

```
1s后主线程释放所
...
t28 running...
t30 running...
强行插入 start...
强行插入 running...
t31 running...
t32 running...
t33 running...
...
```

改为公平锁后：

```java
ReentrantLock lock = new ReentrantLock(true);  // 公平锁
```

强行插入，总是在最后输出：

```
t497 running...
t498 running...
t499 running...
强行插入 running...
```

公平锁一般没有必要，会降低并发度，后面分析原理时会讲解

#### 条件变量

synchronized 中也有条件变量，就是我们讲原理时那个 waitSet 休息室，当条件不满足时进入 waitSet 等待

ReentrantLock 的条件变量比 synchronized 强大之处在于，它是**支持多个条件变量的**，这就好比

- synchronized 是那些不满足条件的线程都在一间休息室等消息
- 而 ReentrantLock 支持多间休息室，有专门等烟的休息室、专门等早餐的休息室、唤醒时也是按休息室来唤醒

使用要点：

- await 前需要获得锁
- await 执行后，会释放锁，进入 conditionObject 等待
- await 的线程被唤醒（或打断、或超时）取重新竞争 lock 锁
- 竞争 lock 锁成功后，从 await 后继续执行

例子：

```java
static ReentrantLock lock = new ReentrantLock();
static Condition waitCigaretteQueue = lock.newCondition();
static Condition waitbreakfastQueue = lock.newCondition();
static volatile boolean hasCigrette = false;
static volatile boolean hasBreakfast = false;

public static void main(String[] args) {

    new Thread(()->{
        try {
            lock.lock();
            while (! hasCigrette){
                try {
                    LoggerUtils.LOGGER.debug("没有烟，休息去了");
                    waitCigaretteQueue.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            LoggerUtils.LOGGER.debug("等到了它的烟");
        }finally {
            lock.unlock();
        }
    }).start();

    new Thread(()->{
        try {
            lock.lock();
            while (! hasBreakfast){
                try {
                    LoggerUtils.LOGGER.debug("没有早饭，休息去了");
                    waitbreakfastQueue.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            LoggerUtils.LOGGER.debug("等到了它的早餐");
        }finally {
            lock.unlock();
        }
    }).start();

    Sleeper.sleep(1);
    sendBreakfast();
    Sleeper.sleep(1);
    sendCigarette();
}

private static void sendCigarette() {
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("送烟来了");
        hasCigrette = true;
        waitCigaretteQueue.signal();
    } finally {
        lock.unlock();
    }
}

private static void sendBreakfast() {
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("送早餐来了");
        hasBreakfast = true;
        waitbreakfastQueue.signal();
    } finally {
        lock.unlock();
    }
}
```

输出：

```java
22:37:17.380 cn.util.LoggerUtils [Thread-0] - 没有烟，休息去了
22:37:17.383 cn.util.LoggerUtils [Thread-1] - 没有早饭，休息去了
22:37:18.163 cn.util.LoggerUtils [main] - 送早餐来了
22:37:18.163 cn.util.LoggerUtils [Thread-1] - 等到了它的早餐
22:37:19.163 cn.util.LoggerUtils [main] - 送烟来了
22:37:19.163 cn.util.LoggerUtils [Thread-0] - 等到了它的烟
```

### 同步模式：顺序控制

#### 固定运行顺序

比如，必须先 2 后 1 打印

**wait&notify 版**

```java
// 用来同步的对象
static Object lock = new Object();
// t2 运行标记， 代表 t2 是否执行过
static boolean t2Runed = false;

public static void main(String[] args) {
    Thread t1 = new Thread(()->{
        synchronized (lock){
            while (!t2Runed){
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("t1 runing");
        }
    }, "t1");

    Thread t2 = new Thread(()->{
        synchronized (lock){
            // 修改运行标记
            t2Runed = true;
            System.out.println("t2 runing");
            lock.notifyAll();
        }
    }, "t2");

    t1.start();
    t2.start();
}
```

**Park&Unpark 版** 

上述可以看到，wait&notify在实现上很麻烦：

- 首先，需要保证先 wait 再 notify，否则 wait 线程永远得不到唤醒。因此使用了『运行标记』来判断该不该 wait
- 第二，如果有些干扰线程错误地 notify 了 wait 线程，条件不满足时还要重新等待，使用了 while 循环来解决此问题
- 最后，唤醒对象上的 wait 线程需要使用 notifyAll，因为『同步对象』上的等待线程可能不止一个

可以使用 LockSupport 类的 park 和 unpark 来简化上面的题目： 

```java
Thread t1 = new Thread(()->{
    // 当没有『许可』时，当前线程暂停运行；有『许可』时，用掉这个『许可』，当前线程恢复运行
    LockSupport.park();
    System.out.println("t1 running...");
}, "t1");

Thread t2 = new Thread(()->{
    System.out.println("t2 running...");
    // 给线程 t1 发放『许可』（多次连续调用 unpark 只会发放一个『许可』）
    LockSupport.unpark(t1);  // 注意这里的t1
}, "t2");

t1.start();
t2.start();
```

park 和 unpark 方法比较灵活，他俩谁先调用，谁后调用无所谓。并且是以线程为单位进行『暂停』和『恢复』，不需要『同步对象』和『运行标记』 

#### 交替输出 

线程 1 输出 a 5 次，线程 2 输出 b 5 次，线程 3 输出 c 5 次。现在要求输出 abcabcabcabcabc 怎么实现 

**wait&notify**

```java
class SyncWaitNotify{

    private int flag;
    private int loopNumber;

    public SyncWaitNotify(int flag, int loopNumber) {
        // 标记:  1-a  2-b  3-c
        this.flag = flag;
        // 循环次数
        this.loopNumber = loopNumber;
    }

    public void print(String str, int waitFlag, int nextFlag){
        for (int i = 0; i < loopNumber; i++) {
            synchronized (this){
                while (this.flag != waitFlag){
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                System.out.print(str);
                this.flag = nextFlag;
                this.notifyAll();
            }
        }
    }
}
```

```java
public static void main(String[] args) {
    SyncWaitNotify syncWaitNotify = new SyncWaitNotify(1, 5);

    new Thread(()->{syncWaitNotify.print("a", 1, 2);}).start();
    new Thread(()->{syncWaitNotify.print("b", 2, 3);}).start();
    new Thread(()->{syncWaitNotify.print("c", 3, 1);}).start();
}
```

**Lock 条件变量版**

```java
public class Test20 {

    public static void main(String[] args) {

        AwaitSignal as = new AwaitSignal(5);
        Condition a = as.newCondition();
        Condition b = as.newCondition();
        Condition c = as.newCondition();

        new Thread(()->{
            as.print("a", a, b);
        }, "t1").start();

        new Thread(()->{
            as.print("b", b, c);
        }, "t2").start();

        new Thread(()->{
            as.print("c", c, a);
        }, "t3").start();

        as.start(a);
    }
}

class AwaitSignal extends ReentrantLock{

    // 循环次数
    private int loopNumber;

    public AwaitSignal(int loopNumber){
        this.loopNumber = loopNumber;
    }

    public void start(Condition first){
        this.lock();
        try {
            System.out.println("start");
            first.signal();
        }finally {
            this.unlock();
        }
    }

    public void print(String str, Condition current, Condition next){
        for (int i = 0; i < loopNumber; i++) {
            this.lock();
            try{
                current.await();
                System.out.print(str);
                next.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                this.unlock();
            }
        }
    }
}
```

> 注意： 
>
> 该实现没有考虑 a，b，c 线程都就绪再开始 

**Park Unpark 版**

```java
public class Test {

    public static void main(String[] args) {
        SyncPark syncPark = new SyncPark(5);
        Thread t1 = new Thread(()->{
            syncPark.print("a");
        });

        Thread t2 = new Thread(()->{
            syncPark.print("b");
        });

        Thread t3 = new Thread(()->{
            syncPark.print("c");
        });

        syncPark.setThreads(t1, t2, t3);
        syncPark.start();
    }
}

class SyncPark{

    private int loopNumber;
    private Thread[] threads;

    public SyncPark(int loopNumber) {
        this.loopNumber = loopNumber;
    }

    public void setThreads(Thread... threads){
        // 创建线程数组
        this.threads = threads;
    }

    public void print(String str){
        for (int i = 0; i < loopNumber; i++) {
            LockSupport.park();
            System.out.print(str);
            LockSupport.unpark(nextThread());
        }
    }

    private Thread nextThread(){
        // 获取当前线程
        Thread current = Thread.currentThread();
        int index = 0;
        for (int i = 0; i < threads.length; i++) {
            // 遍历线程数组
            if(threads[i] == current){
                // 获取当前线程的index
                index = i;
                break;
            }
        }

        // 获取当前线程的下一个线程后返回
        if(index < threads.length - 1){
            return threads[index+1];
        }else{
            return threads[0];
        }
    }

    public void start(){
        // 令所有的线程开启，开启后进入park状态
        for (Thread thread: threads){
            thread.start();
        }

        // 令第一个线程unpark
        LockSupport.unpark(threads[0]);
    }
}
```

### 本章小结

本章我们需要重点掌握的是

- 分析多线程访问共享资源时，哪些代码片段属于临界区
- 使用 synchronized 互斥解决临界区的线程安全问题
  - 掌握 synchronized 锁对象语法
  - 掌握 synchronzied 加载成员方法和静态方法语法
  - 掌握 wait/notify 同步方法
- 使用 lock 互斥解决临界区的线程安全问题
  - 掌握 lock 的使用细节：可打断、锁超时、公平锁、条件变量
- 学会分析变量的线程安全性、掌握常见线程安全类的使用
- 了解线程活跃性问题：死锁、活锁、饥饿
- 应用方面
  - 互斥：使用 synchronized 或 Lock 达到共享资源互斥效果
  - 同步：使用 wait/notify 或 Lock 的条件变量来达到线程间通信效果
- 原理方面
  - monitor、synchronized 、wait/notify 原理
  - synchronized 进阶原理
  - park & unpark 原理
- 模式方面
  - 同步模式之保护性暂停
  - 异步模式之生产者消费者
  - 同步模式之顺序控制

