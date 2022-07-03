# Java：AQS 小记-1(概述)

> AQS 全称是 Abstract Queued Synchronizer（抽象队列同步器），是阻塞式锁和相关的同步器工具的框架，这个类在 `java.util.concurrent.locks` 包下面。
>
> 下面对其作进一步分析。

## 概述

AQS 全称是 Abstract Queued Synchronizer（抽象队列同步器），是阻塞式锁和相关的同步器工具的框架，这个类在 `java.util.concurrent.locks` 包下面。

 **AQS 的核心思想**：

- 如果被请求的共享资源**空闲**，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
- 如果被请求的共享资源**被占用**，那么就需要⼀套**线程阻塞等待**以及**被唤醒时锁分配**的机制，这个机制 AQS 是用 **CLH 队列锁**实现的，即将暂时获取不到锁的线程加入到队列中。

### state 变量

**AQS 使用⼀个 int 类型的成员变量 state 来表示资源（独占/共享）的状态**

```java
// 共享变量，使用volatile修饰保证线程可见性
private volatile int state;
```

state 状态信息通过方法 `getState`， `setState`， `compareAndSetState` 进行获取与修改

```java
// 获取 state 状态
protected final int getState() {
    return state;
}

// 设置 state 状态
protected final void setState(int newState) {
    state = newState;
}

// cas 机制设置state状态
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

**AQS 对资源的占用分为两种方式：Exclusive（独占）与 Share（共享）**

- **Exclusive（独占）**：只有⼀个线程能执行，如 ReentrantLock，又可分为公平锁和非公平锁：
  - **公平锁**：多个线程按照申请锁的顺序去获取锁，线程会先进入队列中进行排队，因此队列中第一个线程会先获得锁资源；
  - **非公平锁**：当线程要获取锁时，会先尝试获得锁，获取不到再进入队列等待（**队列中等待的线程还是遵循FIFO的**）。因此当有新来的线程获取锁时，存在和队列中等待的线程竞争的情况，因此是非公平的。
- **Share（共享）**：多个线程可同时执行，如 Semaphore / CountDownLatch。 

> ReentrantReadWriteLock 可以看成是**组合式**，因为ReentrantReadWriteLock也就是读写锁，允许多个线程同时对某⼀资源进行读。 

### CLH 队列

CLH（Craig + Landin + Hagersten）队列是⼀个虚拟的双向队列（虚拟的双向队列即不存在队列实例，**仅存在结点之间的关联关系**）

AQS 是将每条请求共享资源的线程封装成⼀个 CLH 锁队列的⼀个结点（Node）,来完成获取资源线程的排队工作。

![AQS原理图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251600985.PNG)

### 自定义同步器

不同的自定义同步器争用共享资源的方式不同。**自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可**，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。 

**AQS 底层使用了模板方法模式**

同步器的设计是基于模板方法模式的，如果需要自定义同步器⼀般的方式是这样（模板方法模式很经典的⼀个应用）：

1. 使用者继承 AbstractQueuedSynchronizer 并重写指定的方法。（这些重写方法很简单，无非是对于共享资源 state 的获取和释放）
2. 将 AQS 组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

主要就是子类实现**以下几个模版方法**：

```java
// 该线程是否正在独占资源。只有用到condition才需要去实现它。
protected boolean isHeldExclusively() {
    throw new UnsupportedOperationException();
} 

// 独占方式:尝试获取资源，成功则返回true，失败则返回false。
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}      
// 独占方式:尝试释放资源，成功则返回true，失败则返回false。
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}    

// 共享方式：尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}
// 共享方式：尝试释放资源，成功则返回true，失败则返回false。
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}
```

默认情况下，**每个方法都抛出 UnsupportedOperationException，因此自定义同步器时必须重写要用到的方法。 **AQS 类中的其他方法都是 final ，所以无法被其他类重写，只有这几个方法可以被其他类重写。

⼀般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现 `tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared` 中的⼀种即可。但 AQS 也⽀持自定义同步器同时实现独占和共享两种方式，如 ReentrantReadWriteLock。

### AQS 已有实现

**以 ReentrantLock 为例**：

- state初始化为0，表示未锁定状态。
- A线程`lock()`时，会调用`tryAcquire()`独占该锁并将 state+1。
- 此后，其他线程再`tryAcquire()`时就会失败，直到A线程`unlock()`到state=0（即释放锁）为止，其它线程才有机会获取该锁。
- 当然，释放锁之前， A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。
- 但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

**再以 CountDownLatch 以例**：

- 任务分为 N 个子线程去执行， state也初始化为N（注意N要与线程个数⼀致）。
- 这 N 个子线程是并行执行的，每个子线程执行完后 `countDown()` ⼀次， state-1 （CAS）。
- 等到所有子线程都执行完后(即state=0)，会`unpark()`主调用线程，然后主调用线程就会从`await()`函数返回，继续后续动作。

## 自实现不可重入锁

### 自定义同步器

在上一节中已经提到了如何实现自定义同步器，主要分为两步：

1. 使用者继承 AbstractQueuedSynchronizer 并重写指定的方法。（这些重写方法很简单，无非是对于共享资源 state 的获取和释放）
2. 将 AQS 组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

```java
// 1.继承了AbstractQueuedSynchronizer
final class MySync extends AbstractQueuedSynchronizer{
	
    // 2.实现AQS中的模版方法，这里实现了tryAcquire与tryRelease这对组合
    @Override
    protected boolean tryAcquire(int acquires) {
        if (acquires == 1){
            // cas修改state状态
            if (compareAndSetState(0, 1)) {
                // 加上了锁，并设置owner为当前线程
                setExclusiveOwnerThread(Thread.currentThread());
                // 成功获取锁，返回true
                return true;
            }
        }
        return false;
    }

    @Override
    protected boolean tryRelease(int acquires) {
        if(acquires == 1) {
            if(getState() == 0) {
                throw new IllegalMonitorStateException();
            }
            // 将占用线程清空
            setExclusiveOwnerThread(null);
            setState(0);  // 细节：后加入写屏障，这样前面的修改就会对其他线程可见
            return true;
        }
        return false;
    }

    protected Condition newCondition() {
        return new ConditionObject();
    }

    protected boolean isHeldExclusively(){
        // 是否持有独占锁
        return getState() == 1;
    }
}
```

### 自定义锁

有了自定义同步器，很容易复用 AQS ，实现一个功能完备的自定义锁

```java
// 自定义锁（不可重入锁）
class MyLock implements Lock{
	
    // 自定义的同步器，这里其实可以直接把自定义同步器定义为静态内部类的
    static MySync sync = new MySync();

    @Override
    public void lock() {
        // 尝试加锁，不成功，进入等待队列
        // 这里是调用了AQS的acquire，然后在acquire方法中会调用自定义的实现tryAcquire
        sync.acquire(1);  
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        // 尝试加锁，不成功，进入等待队列，可打断
        sync.acquireInterruptibly(1);
    }

    @Override  
    public boolean tryLock() {
        // 尝试一次，不成功返回，不进入队列
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        // 尝试，不成功，进入等待队列，有时限
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        // 释放锁，这里也是调用了AQS中的release，然后在方法release中再调用自定义实现tryRelease
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
        // 生成条件变量
        return sync.newCondition();
    }
}
```

### 测试

```java
// 创建一个自定义锁
MyLock lock = new MyLock();
new Thread(()->{
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("locking...");
        Sleeper.sleep(1);
    }finally {
        LoggerUtils.LOGGER.debug("unlocking...");
        lock.unlock();
    }
}, "t1").start();

new Thread(()->{
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("locking...");
    }finally {
        LoggerUtils.LOGGER.debug("unlocking...");
        lock.unlock();
    }
}, "t2").start();
```

输出：

```java
11:25:53.601 cn.util.LoggerUtils [t1] - locking...
11:25:54.604 cn.util.LoggerUtils [t1] - unlocking...
11:25:54.604 cn.util.LoggerUtils [t2] - locking...
11:25:54.604 cn.util.LoggerUtils [t2] - unlocking...
```

**不可重入测试**：

如果改为下面代码，会发现自己也会被挡住

```java
new Thread(()->{
    lock.lock();
    LoggerUtils.LOGGER.debug("locking 1...");
    // 只打印上面这个，由于不可重入，不会执行下面的代码
    lock.lock();
    LoggerUtils.LOGGER.debug("locking 2...");
}, "t1").start();
```

## AQS 现有实现概述

大致说明一下，后续会继续对其进行深入分析

### Semaphore

```java
public class Semaphore implements java.io.Serializable {
    // 同步器在这里！！！
    private final Sync sync;
    abstract static class Sync extends AbstractQueuedSynchronizer {
    	// ...
    }
    // 非平锁
    static final class NonfairSync extends Sync {}
    // 公平锁
    static final class FairSync extends Sync {}
```

**Semaphore / 信号量**：允许多个线程同时访问，synchronized 和 ReentrantLock 都是⼀次只允许⼀个线程访问某个资源， Semaphore 可以指定多个线程同时访问某个资源。

> 信号量主要用于两个目的：
>
> - **一个是用于共享资源的互斥使用**
> - **另一个用于并发线程数的控制**
>
> 通过 `new Semaphore(3);` 来指定允许访问线程数，通过 `acquire()` 来获取信号量，通过 `release();` 来释放许可
>
> - `acquire`：通过该方法获取到一个许可，然后对共享资源进行操作；如果许可已经分配完了，那么线程将进入等待状态，直到其他线程释放许可才有机会再次获得许可；
> - `release`：线程释放一个许可，许可将被归还给 Semaphore

### CountDownLatch

```java
public class CountDownLatch {
    // 同步器在这里！！！
    private final Sync sync;
    private static final class Sync extends AbstractQueuedSynchronizer {
		// ...
    }
}
```

**CountDownLatch / 倒计时器**：CountDownLatch 是⼀个同步工具类，用来协调多个线程之间的同步。这个工具通常**用来控制线程等待**，它可以**让某⼀个线程等待直到倒计时结束**，再开始执行。其通过一个计数器来实现，计数器的初值是线程的数量，当每一个线程执行完成后，计数器的值-1，当计数器的值为 0 时，表示所有其他线程都执行完毕，**等待的线程继续执行**。

> 关键方法：
>
> - `await`：调用该方法的线程会被挂起，直到 count 的值为 0 时才继续执行；
> - `countDown`：将 count 值减1

### CyclicBarrier

**CyclicBarrier / 循环栅栏**：CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的计数等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。 CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，**让⼀组线程到达⼀个屏障（也可以叫同步点）时被阻塞，直到最后⼀个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活**。CyclicBarrier 默认的构造方法是 `CyclicBarrier(int parties)`，其参数表示屏障拦截的线程数量，每个线程调用 `await()`方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。 

> 关键方法：`await`：该方法告诉 CyclicBarrier 自己已经达到同步点，然后当前线程被阻塞，该方法有带超时时间和不带超时时间的方法；
>
> 其他：CountDownLatch 基于 AQS 的共享模式；CyclicBarrier 基于 Condition 实现

### ReentrantLock

直接上个类图吧...

![ReentrantLock类图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601223.PNG)

## 参考

https://www.bilibili.com/video/BV16J411h7Rd

http://snailclimb.gitee.io/
