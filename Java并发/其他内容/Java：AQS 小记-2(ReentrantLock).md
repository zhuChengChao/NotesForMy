# Java：AQS 小记-2(ReentrantLock)

> 对AQS的实现类ReentrantLock做进一步分析。

## 整体结构

### ReentrantLock 类图

![ReentrantLock类图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601526.PNG)

### AbstractOwnableSynchronizer 类

```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    protected AbstractOwnableSynchronizer() { }
	
    // 占用资源的线程
    private transient Thread exclusiveOwnerThread;
	
    // 设置占用资源的线程
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
	
    // 获取当前占用资源的线程
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```

### AbstractQueuedSynchronizer 类

```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {

    // ...

    // 内部的 Node 类，组成了 CLH 队列
    static final class Node {
    	// 共享模式下的节点，以共享模式等待锁
        static final Node SHARED = new Node();
        // 独占模式下的节点，以独占的方式等待锁
        static final Node EXCLUSIVE = null;

        // 节点的状态=1：1-线程获取锁的请求已经取消了
        static final int CANCELLED =  1;
        // 节点的状态=-1：线程已经准备好了等待资源释放
        static final int SIGNAL    = -1;
        // 节点的状态=-2：节点在等待条件来唤醒
        static final int CONDITION = -2;
        // 节点的状态=-3：节点在等待条件来唤醒
        static final int PROPAGATE = -3;

    	// Node节点的状态
        volatile int waitStatus;

        // Node的前驱节点，即为双向队列了
        volatile Node prev;

        // Node的后继节点
        volatile Node next;

        // Node是用于封装等待线程的
        volatile Thread thread;

		// ...
	}

	// 队列的头节点
    private transient volatile Node head;

    // 队列的尾节点
    private transient volatile Node tail;

    // 同步资源的状态
    private volatile int state;
}
```

### ReentrantLock类

```java
// ReentrantLock类源代码
public class ReentrantLock implements Lock, java.io.Serializable {

    // ...

    // ReentrantLock类的同步器：
    // Lock接口的实现类，基本都是通过聚合了一个队列同步器的子类完成线程访问控制
    private final Sync sync;
    // 这个同步器Sync继承了AbstractQueuedSynchronizer，且是抽象类用于被继承，后续公平锁和非公平锁就继承了该类
    abstract static class Sync extends AbstractQueuedSynchronizer {
        // 提供了抽象方法，该方法在NonfairSync、FairSync中实现了
        abstract void lock();

        // ☆尝试获取资源，后续分析☆
        final boolean nonfairTryAcquire(int acquires) {
            // 获取当前线程
            final Thread current = Thread.currentThread();
            // 获取state的状态
            int c = getState();
            // 当state的状态为0，说明当前线程可以获取资源
            if (c == 0) {
                // 通过CAS设置资源状态
                if (compareAndSetState(0, acquires)) {
                    // 把现在占用资源的线程设为自身
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                // 可重入的体现
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        // ☆AQS中的模版方法，尝试释放资源，后续分析☆
        protected final boolean tryRelease(int releases) {
            // 获取当前占用资源的线程的state，然后-releases
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            // free体现了可重入，只有state=0时，才为true
            boolean free = false;
            if (c == 0) {
                // 说明state为0了，要释放资源了
                free = true;
                // 释放占用资源的线程
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
		
        // AQS中的模版方法，判断线程是否独占资源
        protected final boolean isHeldExclusively() {
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        // ...
    }

    // 非公平锁的内部静态类实现，继承了Sync
    static final class NonfairSync extends Sync {
        // ...
        // 实现了抽象方法lock
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
		
        // AQS 中的模版方法：和Sync中的tryRelease两个模版方法组成一对
        protected final boolean tryAcquire(int acquires) {
            // 直接调用父类Sync中的nonfairTryAcquire方法
            return nonfairTryAcquire(acquires);
        }
    }

    // 公平锁内部静态类实现，继承了Sync
    static final class FairSync extends Sync {
        // ...
        // 实现了抽象方法lock
        final void lock() {
            acquire(1);
        }
		
        // AQS 中的模版方法：和Sync中的tryRelease两个模版方法组成一对
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                // 和父类的tryAcquire唯一不同之处：hasQueuedPredecessors()多了这个方法
                // 该方法为当等待队列中有线程存在时，返回的是true，则取反为false，则新进来的线程获取锁失败，需要进入等待队列中等待！
                if (!hasQueuedPredecessors() &&  
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
}
```

上述 ReentrantLock 源码的公平锁和非公平锁实现源码中，在获取尝试获取资源的方法中 `tryAcquire` 仅存在一处方法的不同，即为：`hasQueuedPredecessors()`，这个方法用于判断队列中是否有节点，若有节点则返回 true ，然后会处理当前队列中的线程，而将抢占锁的线程放入队列中，见后续分析。

由此引出的公平锁和非公平锁：

- **公平锁**：公平锁讲究先来先到，线程在获取锁时，如果这个锁的等待队列中已经有线程在等待，那么当前线程就会进入等待队列中;
- **非公平锁**：不管是否有等待队列，如果可以获取锁，则立刻占有锁对象。也就是说**队列的第一个排队线程在 `unpark()`之后还是需要竞争锁**（存在线程竞争的情况下)

## 非公平原理 :exclamation:

### 示例代码

```java
// 建立一个非公平锁
ReentrantLock lock = new ReentrantLock();

// 线程A，先抢到锁运行
new Thread(() -> {
    lock.lock();

    try {
        System.out.println(Thread.currentThread().getName()+" come in.");
        try {
            // 办理业务的时间
            TimeUnit.SECONDS.sleep(60);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }finally {
        lock.unlock();
    }
}, "A").start();

// 线程B，由于线程A抢到了锁，会进入等待队列
new Thread(() -> {
    lock.lock();
    try {
        System.out.println(Thread.currentThread().getName()+" come in.");
    }finally {
        lock.unlock();
    }
}, "B").start();

// 线程C，同样由于线程A抢到了锁，会进入等待队列
new Thread(() -> {
    lock.lock();
    try {
        System.out.println(Thread.currentThread().getName()+" come in.");
    }finally {
        lock.unlock();
    }
}, "C").start();
```

### 初始状态

**文字说明**

* ReentrantLock 默认初始化为非公平锁，默认的 state = 0，exclusiveOwnerThread = null，等待线程来获取锁

**图解**

![AQS源码解析1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601702.PNG)

**源码分析**：创建 `new ReentrantLock();`，默认为非公平锁

```java
public ReentrantLock() {
    // 默认实现是非公平锁
    sync = new NonfairSync();
}
```

### 首次获取

**文字说明**

1. 当没有竞争时，线程A成功获取锁；
2. 通过 CAS 将 state 状态设置为 1，
3. 将 exclusiveOwnerThread 设置为自身

**图解**

![AQS源码解析2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601711.PNG)

**源码分析**：**线程A第一次获取资源，成功获取**

```java
// 1.案例代码中获取锁
lock.lock();

// 2.调用ReentrantLock中的方法lock获取锁
public void lock() {
    sync.lock();
}

// 3.由于是非公平锁，因此会调用内部静态类NonfairSync中的方法
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

// 4.执行方法：compareAndSetState(0, 1)，AQS类中的方法
// 期望值为0，设定值为1，第一次判断是，由于AQS的state的确为0，因此CAS成功，将state状态改为1
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}

// 5.在4中cas成功，返回true，执行方法setExclusiveOwnerThread(Thread.currentThread());
// 该方法为AQS的父类AbstractOwnableSynchronizer中的方法，成功将占用线程设置为当前线程A
protected final void setExclusiveOwnerThread(Thread thread) {
    // private transient Thread exclusiveOwnerThread;
    exclusiveOwnerThread = thread;
}
```

### 阻塞获取1

**文字说明**

1. 当第一次竞争出现时，此时线程A已经占用着资源，线程B进入后通过 CAS 尝试将 state 由 0 修改为 1，但是失败；
2. 进入 acquire 函数，通过 tryAcquire 再次尝试获取资源，但是资源还是被线程 A 占用，因此还是失败；
3. 进入 addWaiter 函数：
   1. 构造 Node 节点，需要注意的是，由于此时的队列为 null，因此会先创建一个 **哨兵节点**，此时节点的 waitStatue 都为 0；
   2. 同时将 构造的 Node 节点添加到 哨兵节点 之后；
4. 进入 acquireQueued 方法：
   1. 由于当前节点的前一个节点为哨兵节点，因此还会再次尝试获取锁，当然还是获取失败；
   2. 然后执行方法 shouldParkAfterFailedAcquire，将哨兵节点的 waitState 由0设置为-1，这是会返回false；
   3. 再次尝试获取锁，还是获取失败；
   4. 再次进入 shouldParkAfterFailedAcquire，由于此时哨兵节点的 waitState=-1，因此返回 true，则执行方法 parkAndCheckInterrupt，**该方法中通过 LockSupport.park 阻塞当前线程**！

**图解**

![AQS源码解析3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601554.PNG)

**源码分析**：**线程B获取锁，但是获取失败，进入队列中**

```java
// 1.线程B尝试获取锁
lock.lock();

// 2.调用ReentrantLock中的方法lock获取锁
public void lock() {
    sync.lock();
}

// 3.由于是非公平锁，因此会调用内部静态类NonfairSync中的方法
final void lock() {
    // 执行方法AQS类中的方法compareAndSetState，但是此时A已经占用资源了state=1，因此cas失败，返回false
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        // 上述返回false，则B线程此时进入该出执行
        acquire(1);
}

// 4.执行方法：AQS类中的方法acquire(1)，
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&   // 5.再次尝试获取资源，返回false说明获取失败，取反为true
        // 6.1 执行addWaiter方法; 6.2 执行acquireQueued方法
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

// 5.先执行方法tryAcquire(arg)，这方法是AQS中的模版方法，在ReentrantLock中Sync中对其进行了重写，结构如下：
// ReentrantLock中的静态抽象内部类Sync继承了AQS，而Sync的子类NonfairSync重写了该方法
// ReentrantLock.NonfairSync类中的方法
protected final boolean tryAcquire(int acquires) {
    // 调用了父类Sync中的方法nonfairTryAcquire
    return nonfairTryAcquire(acquires);
}
// Sync抽象静态内部类中的方法nonfairTryAcquire
final boolean nonfairTryAcquire(int acquires) {
    // 获取当前线程
    final Thread current = Thread.currentThread();
    // 获取state的状态
    int c = getState();
    // 当state的状态为0，说明此时刚刚占用资源的线程又释放资源了，则当前线程可以获取资源
    if (c == 0) {
        // 这个流程和上述线程A获取资源的方式一样
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 这里做了一个可重入的判断，即判断当前占用资源的线程是不是就是自身
    else if (current == getExclusiveOwnerThread()) {
        // 如果是的话，则累加state
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    // 返回false说明再次获取资源失败
    return false;
}

// 6.1 执行方法addWaiter(Node.EXCLUSIVE),同样是AQS类中的方法
// 传入的参数为Node.EXCLUSIVE排他的模式的节点
private Node addWaiter(Node mode) {
    // 新建立Node节点，Node节点中就封装了线程B，且是独占模式的Node节点类型
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;  // 由于要尾插，因此由这一步操作，当然第一次时，队列为空，pred=null
    // 第一次往队列中添加节点时并不会进入
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 添加节点如CLH队列
    enq(node);
    // 返回新建立的节点
    return node;
}
// 同样是AQS中的方法
private Node enq(final Node node) {
    for (;;) {  // 死循环
        Node t = tail;
        if (t == null) { // Must initialize
            // 第一次添加时进入，compareAndSetHead后续有分析
            if (compareAndSetHead(new Node()))
                // ☆☆☆第一次有Node节点入队时，会新建一个节点new Node()作为头节点
                // 即在方法compareAndSetHead中传入了一个new Node作为新的头节点
                tail = head;  // 然后让tail也指向这个头节点
        } else {
            // 再次循环到这里，这是B线程构成的节点才真正入队
            node.prev = t;  // 指针指向修改
            // 设置尾结点
            if (compareAndSetTail(t, node)) {
                t.next = node;
                // 返回哨兵节点
                return t;
            }
        }
    }
}
// 比较设置头节点
private final boolean compareAndSetHead(Node update) {
    return unsafe.compareAndSwapObject(this, headOffset, null, update);
}
// 比较设置尾节点
private final boolean compareAndSetTail(Node expect, Node update) {
    return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}

// 6.2 执行acquireQueued方法，传入的参数为当前线程组成的节点
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        // 打断标志位
        boolean interrupted = false;
        for (;;) {  // 死循环
            // 获取当前的节点的前一个节点，由于死循环，一直会获取到哨兵节点
            final Node p = node.predecessor();
            // p节点为投节点 && 当前线程再去尝试获取一下锁（最后的挣扎）false抢失败
            if (p == head && tryAcquire(arg)) {
                // 如果抢成功了，则...
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 执行方法shouldParkAfterFailedAcquire
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            // 当failed为true，则取消排
            cancelAcquire(node);
    }
}
// 执行方法shouldParkAfterFailedAcquire，传入当前线程组成的节点和其前一个节点。
// 这个方法会把当前线程构成的节点在队列中的所有前驱节点的waitStatus都设置为-1
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;  // 获取前一个节点的状态，此时是哨兵节点的状态
    if (ws == Node.SIGNAL)
        // 若当前节点的前一个节点的waitState为-1，则返回true
        // 对于当前B节点而言的话，其前节点为哨兵节点的状态已经为-1，则返回true
        return true;
    if (ws > 0) {
        // 前驱节点被取消
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // pred节点的waitStatue=0 或者是 PROPAGATE=-3，将其改为-1
        // 此时对于B线程而言的话是将哨兵节点的waitStatue的值修改为-1
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
// 当哨兵节点的waitStatus=-1，则进入parkAndCheckInterrupt()
private final boolean parkAndCheckInterrupt() {
    // LockSupport中的park方法对当前线程进行阻塞！这时候才真正进入阻塞
    LockSupport.park(this);
    return Thread.interrupted();
}   
```

### 阻塞获取2

**文字说明**：

* 当资源被占用时，其他线程再次尝试获取资源，都是无法成功获取的，都会进入队列中等待

**图解说明**：

![AQS源码解析4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601473.PNG)

### 解锁流程1

**文字说明：**

1. 占用资源的线程此时释放资源，执行方法 `unlock()`；
2. 进入 release 流程，首先调用 tryRelease 方法：
   1. 通过 CAS 将 state 设置为 0；
   2. 将 exclusiveOwnerThread 设置为 null
3. 当前的队列不为空，获取 哨兵节点，且此时的哨兵节点的 waitState = -1，因此进入 unparkSuccessor 流程；
4. 找到队列中离 哨兵节点 最近的没有取消的 Node，通过 unpark 将其唤醒；
5. 成功唤醒后回到方法 parkAndCheckInterrupt ，然后再回到方法 acquireQueued，然后通过方法 `tryAcquire` 再次去获取资源，获取成功则：
   1. 通过 CAS 将 state 状态设置为 1，
   2. 将 exclusiveOwnerThread 设置为自身
6. 然后更新哨兵节点

**图解：**

![AQS源码解析5](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601724.png)

**源码分析**：**线程A此时要开始解锁了，即线程A执行 `lock.unlock()`**

```java
// 1.线程A执行lock.unlock()
public void unlock() {
    // 调用同步器的release方法
    sync.release(1);
}

// 2.该方法为AQS中的方法
public final boolean release(int arg) {
	// 3.调用方法tryRelease，为AQS中的模版方法，在ReentrantLock中的Sync类中对其进行了实现
    if (tryRelease(arg)) {
        // 现在已经没有线程占用资源了，则先获取头节点
        Node h = head;
        // 头节点不为null，且node的waitStatue为-1，说明有线程在队列中等待
        if (h != null && h.waitStatus != 0)
            // 4.把这个h节点唤醒，即唤醒哨兵节点
            unparkSuccessor(h);
        return true;
    }
    return false;
}

// 3.ReentrantLock.Sync类中对其进行了实现
protected final boolean tryRelease(int releases) {
    // 获取当前占用资源的线程的state，然后-1，当前线程A占用资源，state的确为1
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        // 说明state为0了，要释放资源了
        free = true;
        // 设置占用资源的线程为null
        setExclusiveOwnerThread(null);
    }
    // 更新AQS的state状态
    setState(c);
    // 返回true说明线程资源没有被占用了
    return free;
}

// 4.把这个h节点唤醒，即唤醒哨兵节点
private void unparkSuccessor(Node node) {
    
    // 获取当前哨兵节点的waitStatus
    int ws = node.waitStatus;
    if (ws < 0)
        // 小于0则，设置节点的waitStatus为0
        compareAndSetWaitStatus(node, ws, 0);
	
    // 获取下一个节点
    Node s = node.next;
    // 下一个节点为null || 下一个节点的waitStatus>0
    // 而此时B节点的waitStatus=-1, 因此不满足
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    // 此时s为B节点，不为null
    if (s != null)
        // B节点要被唤醒了，通过s.tread获取B线程对象，然后unpark方法唤醒
        LockSupport.unpark(s.thread);
}

// 5.一旦刚刚park的线程被unpark,则线程恢复运行会跳转到刚刚被阻塞的地方继续执行，即会回到方法parkAndCheckInterrupt
private final boolean parkAndCheckInterrupt() {
    // LockSupport中的park方法对当前线程进行阻塞！这时候才真正进入阻塞
    LockSupport.park(this);
    // 上述通过unpark方法唤醒了线程，则通过方法Thread.interrupted()判断线程是否打断，很显然这里没有被打算，因此返回false
    return Thread.interrupted();
}  

// 6.然后跳出该方法回到其调用者acquireQueued
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        // 打断标志位
        boolean interrupted = false;
        for (;;) {  // 死循环
            final Node p = node.predecessor();
            // 当线程恢复运行后，说明之前占用资源的线程已经释放资源了，然后当前线程去抢占资源
            if (p == head && tryAcquire(arg)) {
                // 线程抢占资源成功
                // 头节点设置为当前抢占资源成功的那个节点，之前是阻塞状态的节点
                setHead(node);
                // 哨兵节点断开连接
                p.next = null; // help GC
                failed = false;
                // 返回线程是否被打算interrupted，到调用者
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 线程在parkAndCheckInterrupt中被阻塞后又恢复运行，返回true
                parkAndCheckInterrupt())
                // 线程恢复运行，打算标志位置位
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### 解锁流程2

**文字说明**：

1. 非公平锁的体现就在此，当占用资源的线程释放锁后；
2. 这时候有其它线程来竞争（非公平的体现），例如这时有 Thread-D 来了；
3. 这个  Thread-D 会和队列的头部节点竞争锁：
   1. 若 Thread-D 竞争成功，则 Thread-4 被设置为 exclusiveOwnerThread，state = 1；而 Thread-B 获取锁失败，重新被 park 阻塞
   2. 若 Thread-D 竞争失败，则 Thread-D 进入队列中

**图解**：

![AQS源码解析6](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601517.PNG)

**源码分析**：

* 略略略...

## 可重入原理

```java
// 拿非公平锁举例
static final class NonfairSync extends Sync {
    // ...
    
    // 注意：这个方法其实是继承自父类Sync，放在这里为了方便查看
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 如果已经获得了锁, 线程还是当前线程, 表示发生了锁重入
        else if (current == getExclusiveOwnerThread()) {
            // state ++
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    
    // 注意：这个方法其实是继承自父类Sync，放在这里为了方便查看
    protected final boolean tryRelease(int releases) {
        // state--
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        // 默认为false，说明当存在重入时，-1之后还不会为0
        boolean free = false;
        // 支持锁重入, 只有 state 减为 0, 才释放成功
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }
}
```

## 可打断原理

由于线程的阻塞是通过 `LockSupport.park()` 方法阻塞，而这个方法其实是可以被打断的。因此后续介绍了 ReentrantLock 的可打断与不可打断的实现方式

### 不可打断模式

在此模式下，

1. **即使它被打断，仍会驻留在 AQS 队列中，继续被park住；**

2. **一直要等到获得锁后方能得知自己被打断了，自己再打断自己一次**

```java
// Sync 继承自 AQS
static final class NonfairSync extends Sync {
    
    // 注意：这个方法是AQS中的方法，而Sync继承了AQS，NonfairSync继承Sync，放在这里方便查看
    private final boolean parkAndCheckInterrupt() {
        // 2.其他线程找到阻塞的线程，对其进行打断
        LockSupport.park(this);  // 当打算标记为true，则park会失效
        // Thread.interrupted()：判断当前线程是否被打断，调用后会清除打断标记，清除打断标记是为了下次park时还可以park住
        // 3.线程被打断，返回true
        return Thread.interrupted();
    }
    
     // 注意：这个方法是AQS中的方法，而Sync继承了AQS，NonfairSync继承Sync，放在这里方便查看
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {  // 6.由于是死循环，因此当线程被打断后还是会进入
                final Node p = node.predecessor();
                // 7.再次尝试获取锁tryAcquire，获取失败则进入1继续被park
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null;
                    failed = false;
                    // 8.获取锁成功，才能返回打断状态
                    return interrupted;
                }
                if (
                    shouldParkAfterFailedAcquire(p, node) &&
                    // 1.线程进入这里被阻塞
                    // 4.当被打断后会返回true
                    parkAndCheckInterrupt()  
                ) {
                    // 5.如果是因为interrupt被唤醒, 返回打断状态为true
                    interrupted = true;
                }
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    
	// 注意：这个方法是AQS中的方法，而Sync继承了AQS，NonfairSync继承Sync，放在这里方便查看
    public final void acquire(int arg) {
        if (
            !tryAcquire(arg) &&
            // 8.被打断后，成功获取到锁，则acquireQueued返回到这里，为true判断成立
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        ) {
            // 9.如果打断状态为 true，
            selfInterrupt();
        }
    }
    
	// 注意：这个方法是AQS中的方法，而Sync继承了AQS，NonfairSync继承Sync，放在这里方便查看
    static void selfInterrupt() {
        // 10.重新产生一次中断！！！
        Thread.currentThread().interrupt();
    }
}
```

### 可打断模式

```java
// ReentrantLock调用方法lockInterruptibly，获取一个可打断的锁
public void lockInterruptibly() throws InterruptedException {
    // 1.内部执行的是Sync类的方法acquireInterruptibly
    sync.acquireInterruptibly(1);
}

static final class NonfairSync extends Sync {
    
	// 注意：这个方法是AQS中的方法，而Sync继承了AQS，NonfairSync继承Sync，放在这里方便查看
    public final void acquireInterruptibly(int arg) throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        // 2.尝试获得锁，获取失败则进入方法doAcquireInterruptibly
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }
    
    // 注意：这个方法是AQS中的方法，而Sync继承了AQS，NonfairSync继承Sync，放在这里方便查看
    // 3.可打断的获取锁流程，打断后直接抛出异常
    private void doAcquireInterruptibly(int arg) throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {  // 同样是一个死循环
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt()) {
                    // 在 park 过程中如果被 interrupt 会进入此
                    // 这时候抛出异常, 而不会再次进入 for (;;)
                    throw new InterruptedException();
                    // 而前面的不可打断模式只是重置了一下标志位
                }
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
}
```

## 公平锁实现原理

```java
static final class FairSync extends Sync {
    
    // 上锁操作
    final void lock() {
        acquire(1);
    }
    
    // 注意：Sync从AQS继承过来的方法, 方便阅读, 放在此处
    public final void acquire(int arg) {
        if (
            !tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        ) {
            selfInterrupt();
        }
    }
    
    // ☆与非公平锁主要区别在于 tryAcquire 方法的实现☆
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 先检查 AQS 队列中是否有前驱节点, 没有才去竞争
            if (!hasQueuedPredecessors() &&  // 和非公平锁的区别就是在此处这个函数hasQueuedPredecessors，当队列中有其他的线程时返回true
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    
    // 注意：Sync从AQS继承过来的方法, 方便阅读, 放在此处
    public final boolean hasQueuedPredecessors() {
        Node t = tail;
        Node h = head;
        Node s;
        // 返回true的条件1：h != t 时表示队列中有 Node
        return h != t &&
            (
            // 条件2.1：(s = h.next) == null 表示队列中还有没有老二
            (s = h.next) == null ||
            // 条件2.2：或者队列中老二线程不是此线程
            s.thread != Thread.currentThread()
        );
    }
}
```

## 条件变量实现原理

### 前置说明

**Condition 接口**

```java
public interface Condition {
    // 省略了其他的一些有等待时间的awit方法
    void await() throws InterruptedException;
    void signal();
    void signalAll();
}
```

ReentrantLock 创建一个条件变量

```java
// 建立一个非公平锁
ReentrantLock lock = new ReentrantLock();
// 创建一个条件变量
Condition condition = lock.newCondition();
// ReentrantLock类内部
public Condition newCondition() {
    // 调用了 Sync中的方法newCondition
    return sync.newCondition();
}

// Sync类中的方法
final ConditionObject newCondition() {
    // 该方法为 AQS类中的内部类ConditionObject的构造函数
    return new ConditionObject();
}
```

进入 AQS 中的内部类 ConditionObject

```java
public class ConditionObject implements Condition, java.io.Serializable {
    
    // 第一个等待的节点，这节点的实现类其实就是AQS中的那个静态内部类
    private transient Node firstWaiter;
    // 最后一个等待的节点
    private transient Node lastWaiter;
	// 构造函数
    public ConditionObject() { }

	// 方法略...主要也是通过ConditionObject这个类维护了一个等待队列！
}
```

**总结**：每个条件变量其实就对应着一个等待队列，其实现类是 ConditionObject 

> 注意：这个等待队列是**单向**的，虽然 ConditionObject 内部使用的 Node 节点类具有双向性，但是没有使用 prev 这个属性

### 图解说明

#### await 流程

开始 Thread-0 持有锁，调用 `await()`，进入 ConditionObject 的 `addConditionWaiter()` 流程，**创建新的 Node 状态为 -2（Node.CONDITION），关联 Thread-0，加入等待队列尾部** 

![条件变量实现原理1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601702.PNG)

接下来进入 AQS 的 `fullyRelease()` 流程，释放同步器上的锁，将 exclusiveOwnerThread 设置为 null，将 state 设置为 0

![条件变量实现原理2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251601082.PNG)

unpark AQS 队列中的下一个节点，竞争锁，假设没有其他竞争线程，那么 Thread-1 竞争成功 

![条件变量实现原理3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251602203.PNG)

park 阻塞 Thread-0 

![条件变量实现原理4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251602820.PNG)

#### signal 流程 

假设 Thread-1 要来唤醒 Thread-0 

![条件变量实现原理5](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251602289.PNG)

进入 ConditionObject 的 `doSignal()` 流程，取得等待队列中第一个 Node，即 Thread-0 所在 Node 

![条件变量实现原理6](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251602909.PNG)

执行 `transferForSignal()` 流程，**将该 Node 加入 AQS 队列尾部**，将 Thread-0 的 waitStatus 改为 0，Thread-3 的 waitStatus 改为 -1 

![条件变量实现原理7](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251602621.PNG)

Thread-1 释放锁，进入 unlock 流程

### 源码说明

待补充...

## 参考

https://www.bilibili.com/video/BV16J411h7Rd?p=238&spm_id_from=pageDriver

https://www.bilibili.com/video/BV1Hy4y1B78T?p=20