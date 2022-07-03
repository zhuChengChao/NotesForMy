# Java：阻塞队列

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

## 1. 概述

### 概念

#### 队列

队列就可以想成是一个数组，从一头进入，一头出去，排队买饭

#### 阻塞队列

BlockingQueue 阻塞队列，排队拥堵，首先它是一个队列，而一个阻塞队列在数据结构中所起的作用大致如下图所示：

![队列](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251605449.png)

线程1往阻塞队列中添加元素，而线程2从阻塞队列中移除元素

- 当阻塞队列是空时，从队列中获取元素的操作将会被阻塞
  
  > 当蛋糕店的柜子空的时候，无法从柜子里面获取蛋糕
- 当阻塞队列是满时，从队列中添加元素的操作将会被阻塞
  
  > 当蛋糕店的柜子满的时候，无法继续向柜子里面添加蛋糕了

也就是说：试图从空的阻塞队列中获取元素的线程将会被阻塞，直到其它线程往空的队列插入新的元素

同理，试图往已经满的阻塞队列中添加新元素的线程会被阻塞，直到其它线程往满的队列中移除一个或多个元素，或者完全清空队列后，使队列重新变得空闲起来

### 为什么要用？

去海底捞吃饭，大厅满了，需要进候厅等待，但是这些等待的客户能够对商家带来利润，因此我们非常欢迎他们阻塞在多线程领域：所谓的阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动唤醒

#### 为什么需要 BlockingQueue

好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切 BlockingQueue 都帮你一手包办了

在 concurrent 包发布以前，在多线程环境下，我们每个程序员都必须自己取控制这些细节，尤其还要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。

### 架构

> 你用过 List 集合类？
>
> ArrayList 集合类熟悉么？
>
> 还用过 CopyOnWriteList  和 BlockingQueue 吗？

BlockingQueue 阻塞队列是属于一个接口，底下有七个实现类

- **ArrayBlockQueue**：由数组结构组成的有界阻塞队列
- **LinkedBlockingQueue**：由链表结构组成的有界（但是默认大小 Integer.MAX_VALUE）的阻塞队列
  
  > 有界，但是界限非常大，相当于无界，可以当成无界
- PriorityBlockQueue：支持优先级排序的无界阻塞队列
- DelayQueue：使用优先级队列实现的延迟无界阻塞队列
- **SynchronousQueue**：不存储元素的阻塞队列，也即单个元素的队列
  
  > 生产一个，消费一个，不存储元素，不消费不生产
- LinkedTransferQueue：由链表结构组成的无界阻塞队列
- LinkedBlockingDeque：由链表结构组成的双向阻塞队列

这里需要掌握的是/常用的是：ArrayBlockQueue、LinkedBlockingQueue、SynchronousQueue

## 2. BlockingQueue 核心方法

| 方法类型 | 抛出异常  | 特殊值   | 阻塞   | 超时                 |
| -------- | --------- | -------- | ------ | -------------------- |
| 插入     | add(e)    | offer(e) | put(e) | offer(e, time, unit) |
| 移除     | remove()  | poll()   | take() | poll(time, unit)     |
| 检查     | element() | peek()   | 不可用 | 不可用               |

| 类型     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 抛出异常 | 当阻塞队列满时：在往队列中 add 插入元素会抛出 `IIIegalStateException：Queue full` <br />当阻塞队列空时：再往队列中 remove 移除元素，会抛出 `NoSuchException` |
| 特殊值   | 插入方法，成功 true，失败 false <br />移除方法：成功返回出队列元素，队列没有就返回空 |
| 一直阻塞 | 当阻塞队列满时，生产者继续往队列里 put 元素，队列会一直阻塞生产线程直到 put 数据or响应中断退出；<br />当阻塞队列空时，消费者线程试图从队列里 take 元素，队列会一直阻塞消费者线程直到队列可用 |
| 超时退出 | 当阻塞队列满时，队里会阻塞生产者线程一定时间，超过限时后生产者线程会退出 |

### 抛出异常组

但执行 `add()` 方法，向已经满的 ArrayBlockingQueue 中添加元素时候，会抛出异常

```java
// 阻塞队列，需要填入默认值
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);

System.out.println(blockingQueue.add("a"));
System.out.println(blockingQueue.add("b"));
System.out.println(blockingQueue.add("c"));
// 已经满了再去添加
System.out.println(blockingQueue.add("XXX"));
```

运行后：

```bash
true
true
true
Exception in thread "main" java.lang.IllegalStateException: Queue full
```

同时如果我们多取出元素的时候，也会抛出异常，我们假设只存储了3个值，但是取的时候，取了四次

```java
// 阻塞队列，需要填入默认值
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
System.out.println(blockingQueue.add("a"));
System.out.println(blockingQueue.add("b"));
System.out.println(blockingQueue.add("c"));

System.out.println(blockingQueue.remove());
System.out.println(blockingQueue.remove());
System.out.println(blockingQueue.remove());
System.out.println(blockingQueue.remove());
```

那么出现异常

```bash
true
true
true
a
b
c
Exception in thread "main" java.util.NoSuchElementException
```

### 布尔类型组

我们使用 `offer()` 的方法，添加元素时候，如果阻塞队列满了后，会返回 false，否者返回 true

同时在取的时候，如果队列已空，那么会返回 null

```java
BlockingQueue blockingQueue = new ArrayBlockingQueue(3);

System.out.println(blockingQueue.offer("a"));
System.out.println(blockingQueue.offer("b"));
System.out.println(blockingQueue.offer("c"));
System.out.println(blockingQueue.offer("d"));

System.out.println(blockingQueue.poll());
System.out.println(blockingQueue.poll());
System.out.println(blockingQueue.poll());
System.out.println(blockingQueue.poll());
```

运行结果：

```bash
true
true
true
false
a
b
c
null
```

### 阻塞队列组

我们使用 `put()` 的方法，添加元素时候，如果阻塞队列满了后，添加消息的线程，会一直阻塞，直到队列元素减少，会被清空，才会唤醒

一般在消息中间件，比如 RabbitMQ 中会使用到，因为需要保证消息百分百不丢失，因此只有让它阻塞

```java
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
blockingQueue.put("a");
blockingQueue.put("b");
blockingQueue.put("c");

System.out.println(blockingQueue.take());
System.out.println(blockingQueue.take());
System.out.println(blockingQueue.take());
System.out.println(blockingQueue.take());
```

同时使用take取消息的时候，如果内容不存在的时候，也会被阻塞

运行结果：

```bash
a
b
c
// ...一直阻塞...
```

### 不见不散组

使用 offer 插入的时候，需要指定时间，如果2秒还没有插入，那么就放弃插入

```java
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
System.out.println(blockingQueue.offer("a", 2L, TimeUnit.SECONDS));
System.out.println(blockingQueue.offer("b", 2L, TimeUnit.SECONDS));
System.out.println(blockingQueue.offer("c", 2L, TimeUnit.SECONDS));
System.out.println(blockingQueue.offer("d", 2L, TimeUnit.SECONDS));
```

同时取的时候也进行判断

```java
System.out.println(blockingQueue.poll(2L, TimeUnit.SECONDS));
System.out.println(blockingQueue.poll(2L, TimeUnit.SECONDS));
System.out.println(blockingQueue.poll(2L, TimeUnit.SECONDS));
System.out.println(blockingQueue.poll(2L, TimeUnit.SECONDS));
```

如果2秒内取不出来，那么就返回 null

运行结果：

```bash
true
true
true
# 等待了2s
false
a
b
c
# 等待了2s
null
```

## 3. SynchronousQueue

SynchronousQueue 没有容量，与其他 BlockingQueue 不同，**SynchronousQueue 是一个不存储的BlockingQueue**，每一个 put 操作必须等待一个 take 操作，否者不能继续添加元素

下面我们测试 SynchronousQueue 添加元素的过程

首先我们创建了两个线程，一个线程用于生产，一个线程用于消费

生产的线程分别 put 了 A、B、C 这三个字段

```java
BlockingQueue<String> blockingQueue = new SynchronousQueue<>();

new Thread(() -> {
    try {       
        System.out.println(Thread.currentThread().getName() + "\t put A ");
        blockingQueue.put("A");
       
        System.out.println(Thread.currentThread().getName() + "\t put B ");
        blockingQueue.put("B");        
        
        System.out.println(Thread.currentThread().getName() + "\t put C ");
        blockingQueue.put("C");        
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}, "t1").start();
```

消费线程使用 take，消费阻塞队列中的内容，并且每次消费前，都等待5秒

```java
new Thread(() -> {
    try {
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        blockingQueue.take();
        System.out.println(Thread.currentThread().getName() + "\t take A ");

        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        blockingQueue.take();
        System.out.println(Thread.currentThread().getName() + "\t take B ");

        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        blockingQueue.take();
        System.out.println(Thread.currentThread().getName() + "\t take C ");

    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}, "t2").start();
```

最后结果输出为：

```bash
t1	 put A 
t2	 take A 

# 5秒后...

t1	 put B 
t2	 take B 

# 5秒后...

t1	 put C 
t2	 take C 
```

我们从最后的运行结果可以看出，每次 t1线程 向队列中添加阻塞队列添加元素后，t1 输入线程就会等待 t2 消费线程，t2 消费后，t2 处于挂起状态，等待 t1 再存入，从而周而复始，**形成一存一取的状态**

## 4. 阻塞队列的用处

### 生产者消费者模式

一个初始值为0的变量，两个线程对其交替操作，一个加1，一个减1，来5轮

关于多线程的操作，我们需要记住下面几句

- **线程---操作---资源类**
- **判断---干活---通知**
- **防止虚假唤醒机制**

我们下面实现一个简单的生产者消费者模式，首先有资源类 ShareData

```java
/**
 * 资源类
 */
class ShareData {

    private int number = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void increment() throws Exception{
        // 同步代码块，加锁
        lock.lock();
        try {
            // 判断
            while(number != 0) {
                // 等待不能生产
                condition.await();
            }
            // 干活
            number++;
            System.out.println(Thread.currentThread().getName() + "\t " + number);
            // 通知 唤醒
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void decrement() throws Exception{
        // 同步代码块，加锁
        lock.lock();
        try {
            // 判断
            while(number == 0) {
                // 等待不能消费
                condition.await();
            }
            // 干活
            number--;
            System.out.println(Thread.currentThread().getName() + "\t " + number);
            // 通知 唤醒
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

里面有一个 number 变量，同时提供了 increment 和 decrement 的方法，分别让 number 加1和减1

但是我们在进行判断的时候，为了防止出现虚假唤醒机制，不能使用 if 来进行判断，而应该使用 while

```java
// 判断
while(number != 0) {
    // 等待不能生产
    condition.await();
}
```

不能使用 if 判断

```java
// 判断
if(number != 0) {
    // 等待不能生产
    condition.await();
}
```

完整代码

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class ShareData{
    private int number = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void increment() throws Exception{
        // 同步代码块，加锁
        lock.lock();
        try {
            // 判断
            while (number!=0){
                // 等待不能生产
                condition.await();
            }
            // 干活
            number++;
            System.out.println(Thread.currentThread().getName() + "\t " + number);
            // 通知 唤醒
            condition.signalAll();
        } catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public void decrement() throws Exception{
        // 同步代码块，加锁
        lock.lock();
        try {
            // 判断
            while (number==0){
                // 等待不能消费
                condition.await();
            }
            // 干活
            number--;
            System.out.println(Thread.currentThread().getName() + "\t " + number);
            // 通知 唤醒
            condition.signalAll();
        } catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}


public class ProdConsumerTraditionDemo {

    public static void main(String[] args) {
        // 高内聚，低耦合
        // 内聚指的是，一个空调，自身带有调节温度高低的方法
        ShareData shareData = new ShareData();

        // t1线程，生产
        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                try {
                    shareData.increment();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }, "t1").start();

        // t2线程，消费
        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                try {
                    shareData.decrement();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }, "t2").start();
    }

}
```

最后运行成功后，我们一个进行生产，一个进行消费

```
t1	 1
t2	 0
t1	 1
t2	 0
t1	 1
t2	 0
t1	 1
t2	 0
t1	 1
t2	 0
```

### 生成者和消费者 3.0

在 concurrent 包发布以前，在多线程环境下，我们每个程序员都必须自己去控制这些细节，尤其还要兼顾效率和线程安全，则这会给我们的程序带来不小的时间复杂度

现在我们使用新版的阻塞队列版生产者和消费者，使用：volatile、CAS、atomicInteger、BlockQueue、线程交互、原子引用

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

class MyResource{
    // 默认开启，进行生产消费
    // 这里用到了volatile是为了保持数据的可见性，也就是当FLAG修改时，要马上通知其它线程进行修改
    private volatile boolean FLAG = true;

    // 使用原子包装类，而不用number++
    private AtomicInteger atomicInteger = new AtomicInteger();

    // 这里不能为了满足条件，而实例化一个具体的 SynchronousBlockingQueue
    BlockingQueue<String> blockingQueue = null;

    // 而应该采用依赖注入里面的，构造注入方法传入
    public MyResource(BlockingQueue<String> blockingQueue) {
        this.blockingQueue = blockingQueue;
        // 查询出传入的class是什么
        System.out.println(blockingQueue.getClass().getName());
    }

    /**
     * 生产
     * @throws Exception
     */
    public void myProd() throws Exception{
        String data = null;
        boolean retValue;
        // 多线程环境的判断，一定要使用while进行，防止出现虚假唤醒
        // 当FLAG为true的时候，开始生产
        while (FLAG){
            data = atomicInteger.incrementAndGet() + "";

            // 2秒存入1个data，超时则插入失败
            retValue = blockingQueue.offer(data, 2l, TimeUnit.SECONDS);
            if(retValue){
                System.out.println(Thread.currentThread().getName() + "\t 插入队列:" + data  + "成功" );
            }else{
                System.out.println(Thread.currentThread().getName() + "\t 插入队列:" + data  + "失败" );
            }

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println(Thread.currentThread().getName() + "\t 停止生产，表示FLAG=false，生产结束");
    }

    /**
     * 消费
     * @throws Exception
     */
    public void myConsumer() throws Exception{
        String retValue;
        // 多线程环境的判断，一定要使用while进行，防止出现虚假唤醒
        // 当FLAG为true的时候，开始生产
        while (FLAG){
            // 2秒消费1个data
            retValue = blockingQueue.poll(2l, TimeUnit.SECONDS);
            if(retValue != null && retValue != ""){
                System.out.println(Thread.currentThread().getName() + "\t 消费队列:" + retValue  + "成功" );
            }else {
                FLAG = false;
                System.out.println(Thread.currentThread().getName() + "\t 消费失败，队列中已为空，退出" );
            }
        }
    }

    /**
     * 停止生产的判断
     */
    public void stop() {
        this.FLAG = false;
    }
}

public class ProdConsumerBlockingQueueDemo {

    public static void main(String[] args) {
        // 传入具体的实现类， ArrayBlockingQueue
        MyResource myResource = new MyResource(new ArrayBlockingQueue<String>(10));

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t 生产线程启动 \n\n");
            try {
                myResource.myProd();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "prod").start();

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t 消费线程启动");

            try {
                myResource.myConsumer();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "consumer").start();

        // 5秒后，停止生产和消费
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("");
        System.out.println("5秒中后，生产和消费线程停止，线程结束");
        myResource.stop();
    }
}
```

最后运行结果

```bash
java.util.concurrent.ArrayBlockingQueue
prod	 生产线程启动 


consumer	 消费线程启动
prod	 插入队列:1成功
consumer	 消费队列:1成功
prod	 插入队列:2成功
consumer	 消费队列:2成功
prod	 插入队列:3成功
consumer	 消费队列:3成功
prod	 插入队列:4成功
consumer	 消费队列:4成功
prod	 插入队列:5成功
consumer	 消费队列:5成功

5秒中后，生产和消费线程停止，线程结束
prod	 停止生产，表示FLAG=false，生产结束
consumer	 消费失败，队列中已为空，退出

Process finished with exit code 0
```

## 5. 补充说明：Synchronized和Lock的区别

### 概述

早期的时候我们对线程的主要操作为：

- **synchronized；wait；notify**

然后后面出现了替代方案

- **lock；await；singal**

![线程同步方式](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251605580.png)

### 问题

synchronized 和 lock 有什么区别？用新的lock有什么好处？举例说明

1. synchronized 属于 **JVM 层面**，属于 Java 的关键字
   * monitorenter（底层是通过 monitor 对象来完成，其实 wait/notify 等方法也依赖于 monitor 对象 只能在同步块或者方法中才能调用 wait/notify 等方法）
   * Lock是具体类（`java.util.concurrent.locks.Lock`）是 **API 层面**的锁

2. 使用方法：
   * synchronized：不需要用户去手动释放锁，当 synchronized 代码执行后，**系统会自动让线程释放对锁**的占用
   * ReentrantLock：则需要用户去**手动释放锁**，若没有主动释放锁，就有可能出现死锁的现象，需要`lock()` 和 `unlock()` 配置 `try catch finally` 语句来完成

3. 等待是否中断

   * synchronized：**不可中断**，除非抛出异常或者正常运行完成

   * ReentrantLock：**可中断，可以设置超时方法**
     * 设置**超时方法**，`lock.trylock(long timeout, TimeUnit unit)`
     * `lock.lockInterruptibly()` 放代码块中，调用`线程.interrupt()`方法**可以中断**

4. 加锁是否公平

   * synchronized：非公平锁

   * ReentrantLock：默认非公平锁，构造函数可以传递 boolean 值，为 true 则为公平锁，false 为非公平锁

5. 锁绑定多个条件 Condition

   * synchronized：没有，要么随机，要么全部唤醒

   * ReentrantLock：用来实现分组唤醒需要唤醒的线程，**可以精确唤醒**，而不是像 synchronized 那样，要么随机，要么全部唤醒

### 举例

针对刚刚提到的区别的第5条，我们有下面这样的一个场景

```
题目：多线程之间按顺序调用，实现 A->B->C 三个线程启动，要求如下：
AA打印5次，BB打印10次，CC打印15次
紧接着
AA打印5次，BB打印10次，CC打印15次
..
来10轮
```

我们会发现，这样的场景在使用 synchronized 来完成的话，会非常的困难，但是使用 lock 就非常方便了

也就是我们需要实现一个链式唤醒的操作

当 A线程 执行完后，B线程 才能执行，然后 B线程 执行完成后，C线程 才执行

首先我们需要创建一个重入锁

```java
// 创建一个重入锁
private Lock lock = new ReentrantLock();
```

然后定义三个条件，也可以称为锁的钥匙，通过它就可以获取到锁，进入到方法里面

```java
// 这三个相当于备用钥匙
private Condition condition1 = lock.newCondition();
private Condition condition2 = lock.newCondition();
private Condition condition3 = lock.newCondition();
```

然后开始记住锁的三部曲： 判断---干活---唤醒

这里的判断，为了避免虚假唤醒，一定要采用 while

干活就是把需要的内容，打印出来

唤醒的话，就是修改资源类的值，然后精准唤醒线程进行干活：

1. 线程A 唤醒 线程B
2. 线程B 唤醒 线程C
3. 线程C 又唤醒 线程A

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class ShareResource{
    // A 1   B 2   c 3
    private int number = 1;
    // 创建一个重入锁
    private Lock lock = new ReentrantLock();

    // 这三个相当于备用钥匙
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    public void print5() {
        lock.lock();
        try {
            // 判断
            while(number != 1) {
                // 不等于1，需要等待
                condition1.await();
            }

            // 干活
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "\t " + number + "\t" + i);
            }

            // 唤醒 （干完活后，需要通知线程2执行）
            number = 2;
            // 通知2号去干活了
            condition2.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void print10() {
        lock.lock();
        try {
            // 判断
            while(number != 2) {
                // 不等于1，需要等待
                condition2.await();
            }

            // 干活
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "\t " + number + "\t" + i);
            }

            // 唤醒 （干完活后，需要通知线程3执行）
            number = 3;
            // 通知3号去干活了
            condition3.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void print15() {
        lock.lock();
        try {
            // 判断
            while(number != 3) {
                // 不等于1，需要等待
                condition3.await();
            }

            // 干活
            for (int i = 0; i < 15; i++) {
                System.out.println(Thread.currentThread().getName() + "\t " + number + "\t" + i);
            }

            // 唤醒 （干完活后，需要通知线程1执行）
            number = 1;
            // 通知1号去干活了
            condition1.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

public class SyncAndReentrantLockDemo {

    public static void main(String[] args) {

        ShareResource shareResource = new ShareResource();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.print5();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.print10();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.print15();
            }
        }, "C").start();
    }
}
```

