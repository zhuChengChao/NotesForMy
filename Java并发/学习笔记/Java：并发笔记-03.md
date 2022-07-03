# Java：并发笔记-03

> 说明：观看了 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444/)** 的课程 [java并发编程](https://www.bilibili.com/video/BV16J411h7Rd) 后所作的学习笔记，总计拆分为9篇笔记，此为3/9。

## 3. 共享模型之管程-2

### 本章内容-2

- Monitor
- wait/notify

### 3.6 Monitor 概念

#### Java 对象头

以 32 位虚拟机为例

* 普通对象

![普通对象对象头](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546767.PNG)

* 数组对象

![数组对象](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546826.PNG)

* 其中 Mark Word 结构为

![MarkWord结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546907.PNG)

64 位虚拟机 Mark Word 

![MarkWord结构64位](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546277.PNG)

> 参考资料 
>
> https://stackoverflow.com/questions/26357186/what-is-in-java-object-header 

#### 原理：Monitor(锁)

Monitor 被翻译为**监视器**或**管程** 

每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 中就被设置指向 Monitor 对象的指针 

Monitor 结构如下：

![Monitor](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546166.PNG)

- 刚开始 Monitor 中 Owner 为 null
- 当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一个 Owner
- 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行synchronized(obj)，就会进入EntryList BLOCKED
- Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，竞争的时是非公平的
- 图中 WaitSet 中的 Thread-0，Thread-1 是之前获得过锁，但条件不满足进入 WAITING 状态的线程，后面讲wait-notify 时会分析 

> 注意：
>
> - synchronized 必须是进入同一个对象的 monitor 才有上述的效果
> - 不加 synchronized 的对象不会关联监视器，不遵从以上规则

#### 原理： synchronized 基础

```java
static final Object lock = new Object();
static int counter = 0;
public static void main(String[] args) {
    synchronized (lock) {
        counter++;
    }
}
```

对应的字节码为：

```java
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
    	stack=2, locals=3, args_size=1
            0: getstatic #2 		 // <- lock引用 （synchronized开始）
            3: dup
            4: astore_1 			 // lock引用 -> slot 1
            5: monitorenter 		 // 将lock对象 MarkWord 置为 Monitor 指针
            6: getstatic #3			 // <- i
            9: iconst_1				 // 准备常数 1
            10: iadd 			     // +1
            11: putstatic #3  	     // -> i
            14: aload_1 		   	 // <- lock引用
            15: monitorexit          // 将lock对象 MarkWord 重置, 唤醒 EntryList
            16: goto 24
            19: astore_2 			 // e -> slot 2
            20: aload_1              // <- lock引用
            21: monitorexit          // 将 lock对象 MarkWord 重置, 唤醒 EntryList
            22: aload_2              // <- slot 2 (e)
            23: athrow               // throw e
            24: return
    	Exception table:
    		from to target type
    			6 16 19 any
                19 22 19 any
    	LineNumberTable:
    		line 8: 0
    		line 9: 6
    		line 10: 14
    		line 11: 24
    	LocalVariableTable:
    		Start Length Slot Name Signature
    		0 25 0 args [Ljava/lang/String;
    	StackMapTable: number_of_entries = 2
    		frame_type = 255 /* full_frame */
    		offset_delta = 19
    		locals = [ class "[Ljava/lang/String;", class java/lang/Object ]
    		stack = [ class java/lang/Throwable ]
    		frame_type = 250 /* chop */
    		offset_delta = 4
```

> 注意
>
> 方法级别的 synchronized 不会在字节码指令中有所体现 

#### 小故事

故事角色

- 老王 - JVM
- 小南 - 线程
- 小女 - 线程
- 房间 - 对象
- 房间门上 - 防盗锁 - Monitor
- 房间门上 - 小南书包 - 轻量级锁
- 房间门上 - 刻上小南大名 - 偏向锁
- 批量重刻名 - 一个类的偏向锁撤销到达 20 阈值
- 不能刻名字 - 批量撤销该类对象的偏向锁，设置该类不可偏向

故事继续：

* 小南要使用房间保证计算不被其它人干扰（原子性），最初，他用的是防盗锁，当上下文切换时，锁住门。这样，即使他离开了，别人也进不了门，他的工作就是安全的。

* 但是，很多情况下没人跟他来竞争房间的使用权。小女是要用房间，但使用的时间上是错开的，小南白天用，小女晚上用。每次上锁太麻烦了，有没有更简单的办法呢？

* 小南和小女商量了一下，约定不锁门了，而是谁用房间，谁把自己的书包挂在门口，但他们的书包样式都一样，因此每次进门前得翻翻书包，看课本是谁的，如果是自己的，那么就可以进门，这样省的上锁解锁了。万一书包不是自己的，那么就在门外等，并通知对方下次用锁门的方式。

* 后来，小女回老家了，很长一段时间都不会用这个房间。小南每次还是挂书包，翻书包，虽然比锁门省事了，但仍然觉得麻烦。

* 于是，小南干脆在门上刻上了自己的名字：【小南专属房间，其它人勿用】，下次来用房间时，只要名字还在，那么说明没人打扰，还是可以安全地使用房间。如果这期间有其它人要用这个房间，那么由使用者将小南刻的名字擦掉，升级为挂书包的方式。

* 同学们都放假回老家了，小南就膨胀了，在 20 个房间刻上了自己的名字，想进哪个进哪个。后来他自己放假回老家了，这时小女回来了（她也要用这些房间），结果就是得一个个地擦掉小南刻的名字，升级为挂书包的方式。老王觉得这成本有点高，提出了一种批量重刻名的方法，他让小女不用挂书包了，可以直接在门上刻上自己的名字。

* 后来，刻名的现象越来越频繁，老王受不了了：算了，这些房间都不能刻名了，只能挂书包

### 原理：synchronized 进阶

#### 轻量级锁

轻量级锁的使用场景：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化。

**轻量级锁对使用者是透明的**，即语法仍然是 synchronized

假设有两个方法同步块，利用同一个对象加锁 

```java
static final Object obj = new Object();
public static void method1() {
    synchronized( obj ) {
        // 同步块 A
        method2();
    }
}
public static void method2() {
    synchronized( obj ) {
        // 同步块 B
    }
}
```

* 创建锁记录（Lock Record）对象，每个线程的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的Mark Word 

  ![锁记录对象1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546742.PNG)

* 让锁记录中 Object reference 指向锁对象，并尝试用 cas（compare and swap） 替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录

  ![锁记录对象2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546007.PNG)

* 如果 cas 替换成功，对象头中存储了锁记录地址和状态 00 (Lightweight Locked)，表示由该线程给对象加锁，这时图示如下

  ![锁记录对象3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546600.PNG)

* 如果 cas 失败，有两种情况 

  - 如果是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入锁膨胀过程

  - 如果是自己执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数 

    ![锁记录对象4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546312.PNG)

* 当退出 synchronized 代码块（解锁时）如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示重入计数减一

  ![锁记录对象5](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546517.PNG)

* 当退出 synchronized 代码块（解锁时）锁记录的值不为 null，这时使用 cas 将 Mark Word 的值恢复给对象头

  - 成功，则解锁成功
  - 失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程 

#### 锁膨胀

如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁。

```java
static Object obj = new Object();
public static void method1() {
    synchronized( obj ) {
        // 同步块
    }
}
```

- 当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁 

  ![锁膨胀1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546393.PNG)

- 这时 Thread-1 加轻量级锁失败，进入锁膨胀流程

  - 即为 Object 对象申请 Monitor 锁，让 Object 指向重量级锁地址

  - 然后自己进入 Monitor 的 EntryList BLOCKED 

    ![锁膨胀2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546836.PNG)

- 当 Thread-0 退出同步块解锁时，使用 cas 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级解锁流程，即按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 线程 

#### 自旋优化 

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞。

自旋重试成功的情况 

| 线程 1 （core 1 上）     | 对象 Mark              | 线程 2 （core 2 上）     |
| ------------------------ | ---------------------- | ------------------------ |
| -                        | 10（重量锁）           | -                        |
| 访问同步块，获取 monitor | 10（重量锁）重量锁指针 | -                        |
| 成功（加锁）             | 10（重量锁）重量锁指针 | -                        |
| 执行同步块               | 10（重量锁）重量锁指针 | -                        |
| 执行同步块               | 10（重量锁）重量锁指针 | 访问同步块，获取 monitor |
| 执行同步块               | 10（重量锁）重量锁指针 | 自旋重试                 |
| 执行完毕                 | 10（重量锁）重量锁指针 | 自旋重试                 |
| 成功（解锁）             | 01（无锁）             | 自旋重试                 |
| -                        | 10（重量锁）重量锁指针 | 成功（加锁）             |
| -                        | 10（重量锁）重量锁指针 | 执行同步块               |
| -                        | ...                    | ...                      |

自旋重试失败的情况 

| 线程 1（core 1 上）      | 对象 Mark              | 线程 2（core 2 上）      |
| ------------------------ | ---------------------- | ------------------------ |
| -                        | 10（重量锁）           | -                        |
| 访问同步块，获取 monitor | 10（重量锁）重量锁指针 | -                        |
| 成功（加锁）             | 10（重量锁）重量锁指针 | -                        |
| 执行同步块               | 10（重量锁）重量锁指针 | -                        |
| 执行同步块               | 10（重量锁）重量锁指针 | 访问同步块，获取 monitor |
| 执行同步块               | 10（重量锁）重量锁指针 | 自旋重试                 |
| 执行同步块               | 10（重量锁）重量锁指针 | 自旋重试                 |
| 执行同步块               | 10（重量锁）重量锁指针 | 自旋重试                 |
| 执行同步块               | 10（重量锁）重量锁指针 | 阻塞                     |
| -                        | ...                    | ...                      |

- 自旋会占用 CPU 时间，单核 CPU 自旋就是浪费，多核 CPU 自旋才能发挥优势。
- 在 Java 6 之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能。
- Java 7 之后不能控制是否开启自旋功能 

#### 偏向锁

轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 CAS 操作。

Java 6 中引入了偏向锁来做进一步优化：只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有

例如： 

```java
static final Object obj = new Object();
public static void m1() {
    synchronized( obj ) {
        // 同步块 A
        m2();
    }
}
public static void m2() {
    synchronized( obj ) {
        // 同步块 B
        m3();
    }
}
public static void m3() {
    synchronized( obj ) {
        // 同步块 C
    }
}
```

![偏向锁](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546367.PNG)

##### 偏向状态

回忆一下对象头格式

![MarkWord结构64位](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546922.PNG)

一个对象创建时：

- 如果开启了偏向锁（默认开启），那么对象创建后，markword 值为 0x05 即最后 3 位为 101，这时它的thread、epoch、age 都为 0
- 偏向锁是默认是延迟的，不会在程序启动时立即生效，如果想避免延迟，可以加 VM 参数-XX:BiasedLockingStartupDelay=0 来禁用延迟
- 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 hashcode、age 都为 0，第一次用到 hashcode 时才会赋值

测试说明：

1）测试延迟特性

2）测试偏向锁 

```java
class Dog{}  // 对狗这个对象进行分析
```

利用 jol 第三方工具来查看对象头信息（注意这里我扩展了 jol 让它输出更为简洁） 

```java
public static void main(String[] args) throws IOException {
    Dog d = new Dog();
    log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true)});
   	// 输出：由于有延时导致偏向锁还没有开启001
    // 0000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001
    Thread.sleep(4000);
    log.debug(ClassLayout.parseInstance(new Dog()).toPrintableSimple(true)});
    // 偏向锁生效了此时输出：101
    // 0000000 00000000 00000000 00000000 00000000 00000000 00000000 00000101
}
```

为了避免延时：添加虚拟机参数 `-XX:BiasedLockingStartupDelay=0`

```java
public static void main(String[] args) throws IOException {
    Dog d = new Dog();
    ClassLayout classLayout = ClassLayout.parseInstance(d);

    new Thread(() -> {
        log.debug("synchronized 前");
        System.out.println(classLayout.toPrintableSimple(true));
        synchronized (d) {
            log.debug("synchronized 中");
            System.out.println(classLayout.toPrintableSimple(true));
        }
        log.debug("synchronized 后");
        System.out.println(classLayout.toPrintableSimple(true));
    }, "t1").start();
}
```

输出：

```java
11:08:58.117 c.TestBiased [t1] - synchronized 前 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000101
11:08:58.121 c.TestBiased [t1] - synchronized 中 00000000 00000000 00000000 00000000 00011111 11101011 11010000 00000101  // 看！进入synchronized了，有线程id了
11:08:58.121 c.TestBiased [t1] - synchronized 后 00000000 00000000 00000000 00000000 00011111 11101011 11010000 00000101  // 处于偏向锁的对象解锁后，线程 id 仍存储于对象头中
```

> **注意**
>
> 处于偏向锁的对象解锁后，线程 id 仍存储于对象头中 

3）测试禁用 

在上面测试代码运行时在添加 VM 参数 `-XX:-UseBiasedLocking` 禁用偏向锁 

输出：

```java
11:13:10.018 c.TestBiased [t1] - synchronized 前 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001  // 001:noraml，且没有hashcode
11:13:10.021 c.TestBiased [t1] - synchronized 中 00000000 00000000 00000000 00000000 00100000 00010100 11110011 10001000  // 000:轻量锁，生成了hashcode
11:13:10.021 c.TestBiased [t1] - synchronized 后 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001
```

4）测试 hashCode 

- 正常状态对象一开始是没有 hashCode 的，第一次调用才生成 

##### 撤销偏向锁：调用对象 hashCode

```java
Dog d = new Dog();
// 调用了对象的 hashCode
d.hashCode();
```

调用了对象的 hashCode，但偏向锁的对象 MarkWord 中存储的是线程 id，如果调用 hashCode 会导致偏向锁被撤销

- 轻量级锁会在锁记录中记录 hashCode
- 重量级锁会在 Monitor 中记录 hashCode

在调用 hashCode 后使用偏向锁，记得去掉 `-XX:-UseBiasedLocking`

输出：

```java
11:22:10.386 c.TestBiased [main] - 调用 hashCode:1778535015
11:22:10.391 c.TestBiased [t1] - synchronized 前 00000000 00000000 00000000 01101010 00000010 01001010 01100111 00000001  // 001:noraml
11:22:10.393 c.TestBiased [t1] - synchronized 中 00000000 00000000 00000000 00000000 00100000 11000011 11110011 01101000
11:22:10.393 c.TestBiased [t1] - synchronized 后 00000000 00000000 00000000 01101010 00000010 01001010 01100111 00000001
```

##### 撤销偏向锁：其它线程使用对象 

当有其它线程使用偏向锁对象时，会将偏向锁升级为轻量级锁 

```java
private static void test2() throws InterruptedException {
    Dog d = new Dog();
    Thread t1 = new Thread(() -> {
        synchronized (d) {
            log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
        }
        synchronized (TestBiased.class) {
            TestBiased.class.notify();
        }
        // 如果不用 wait/notify 使用 join 必须打开下面的注释
        // 因为：t1 线程不能结束，否则底层线程可能被 jvm 重用作为 t2 线程，底层线程 id 是一样的
        /*try {
        System.in.read();
        } catch (IOException e) {
        e.printStackTrace();
        }*/
    }, "t1");
    t1.start();
    
    Thread t2 = new Thread(() -> {
        synchronized (TestBiased.class) {
            try {
                TestBiased.class.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
        synchronized (d) {
            log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
        }
        // t2使用过了dog对象
        log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
    }, "t2");
    t2.start();
}
```

输出：

```java
[t1] - 00000000 00000000 00000000 00000000 00011111 01000001 00010000 00000101
[t2] - 00000000 00000000 00000000 00000000 00011111 01000001 00010000 00000101
[t2] - 00000000 00000000 00000000 00000000 00011111 10110101 11110000 01000000  // 不加偏向锁了，此时加了轻量级锁
[t2] - 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001
```

##### 撤销偏向锁：调用 wait/notify 

因为wait和notify只有重量级锁才有

```java
public static void main(String[] args) throws InterruptedException {
    Dog d = new Dog();
    Thread t1 = new Thread(() -> {
        log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
        synchronized (d) {
            log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
            try {
                d.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
        }
    }, "t1");
    t1.start();
    
    new Thread(() -> {
        try {
            Thread.sleep(6000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        synchronized (d) {
            log.debug("notify");
            d.notify();
        }
    }, "t2").start();
}
```

输出：

```java
[t1] - 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000101
[t1] - 00000000 00000000 00000000 00000000 00011111 10110011 11111000 00000101
[t2] - notify
[t1] - 00000000 00000000 00000000 00000000 00011100 11010100 00001101 11001010
```

##### 批量重偏向

如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1 的对象仍有机会重新偏向 T2，重偏向会重置对象的 Thread ID

当撤销偏向锁阈值超过 20 次后，jvm 会这样觉得，我是不是偏向错了呢，于是会在给这些对象加锁时重新偏向至加锁线程 

```java
private static void test3() throws InterruptedException {
    Vector<Dog> list = new Vector<>();
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 30; i++) {
            Dog d = new Dog();
            list.add(d);
            synchronized (d) {
                log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
            }
        }
        synchronized (list) {
            list.notify();
        }
    }, "t1");
    t1.start();
    Thread t2 = new Thread(() -> {
        synchronized (list) {
            try {
                list.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        log.debug("===============> ");
        for (int i = 0; i < 30; i++) {
            Dog d = list.get(i);
            log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
            synchronized (d) {
                log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
            }
            log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
        }
    }, "t2");
    t2.start();
}
```

输出：

```java
[t1] - 0 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101
[t1] - 1 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101
// ....
[t1] - 28 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101
[t1] - 29 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101
[t2] - ===============>
[t2] - 0 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101
[t2] - 0 00000000 00000000 00000000 00000000 00100000 01011000 11110111 00000000
[t2] - 0 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001
[t2] - 1 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101
[t2] - 1 00000000 00000000 00000000 00000000 00100000 01011000 11110111 00000000
[t2] - 1 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001
// ...
[t2] - 18 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101
[t2] - 18 00000000 00000000 00000000 00000000 00100000 01011000 11110111 00000000
[t2] - 18 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001
[t2] - 19 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101 // 重新偏向了!!!
[t2] - 19 00000000 00000000 00000000 00000000 00011111 11110011 11110001 00000101  
[t2] - 19 00000000 00000000 00000000 00000000 00011111 11110011 11110001 00000101
// ...
[t2] - 29 00000000 00000000 00000000 00000000 00011111 11110011 11100000 00000101
[t2] - 29 00000000 00000000 00000000 00000000 00011111 11110011 11110001 00000101
[t2] - 29 00000000 00000000 00000000 00000000 00011111 11110011 11110001 00000101
```

##### 批量撤销

当撤销偏向锁阈值超过 40 次后，jvm 会这样觉得，自己确实偏向错了，根本就不该偏向。于是整个类的所有对象都会变为不可偏向的，新建的对象也是不可偏向的

```java
static Thread t1,t2,t3;
private static void test4() throws InterruptedException {
    Vector<Dog> list = new Vector<>();
    int loopNumber = 39;
    t1 = new Thread(() -> {
        for (int i = 0; i < loopNumber; i++) {
            Dog d = new Dog();
            list.add(d);
            synchronized (d) {
                log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
            }
        }
        LockSupport.unpark(t2);
    }, "t1");
    t1.start();
    t2 = new Thread(() -> {
        LockSupport.park();
        log.debug("===============> ");
        for (int i = 0; i < loopNumber; i++) {
            Dog d = list.get(i);
            log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
            synchronized (d) {
                log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
            }
            log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
        }
        LockSupport.unpark(t3);
    }, "t2");
    t2.start();
    t3 = new Thread(() -> {
        LockSupport.park();
        log.debug("===============> ");
        for (int i = 0; i < loopNumber; i++) {
            Dog d = list.get(i);
            log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
            synchronized (d) {
                log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
            }
            log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
        }
    }, "t3");
    t3.start();
    t3.join();
    log.debug(ClassLayout.parseInstance(new Dog()).toPrintableSimple(true));
}
```

> **参考资料：**
>
> https://github.com/farmerjohngit/myblog/issues/12
>
> https://www.cnblogs.com/LemonFive/p/11246086.html
>
> https://www.cnblogs.com/LemonFive/p/11248248.html
>
> 偏向锁论文

#### 锁消除

```java
@Fork(1)
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations=3)
@Measurement(iterations=5)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class MyBenchmark {
    static int x = 0;
    @Benchmark
    public void a() throws Exception {
        x++;
    }
    @Benchmark
    public void b() throws Exception {
        Object o = new Object();
        synchronized (o) {
            x++;
        }
    }
}
```

`java -jar benchmarks.jar `

```java
// 被锁消除后的结果：
Benchmark Mode Samples Score Score error Units
c.i.MyBenchmark.a avgt 5 1.542 0.056 ns/op
c.i.MyBenchmark.b avgt 5 1.518 0.091 ns/op
// 上述结果显示：性能相近，这是由于被Java发现x对象不需要加锁，被JIT（即时编译）优化了，即被锁消除了？？？
```

`java -XX:-EliminateLocks -jar benchmarks.jar `

```java
// 不需要锁消除的结果
Benchmark Mode Samples Score Score error Units
c.i.MyBenchmark.a avgt 5 1.507 0.108 ns/op
c.i.MyBenchmark.b avgt 5 16.976 1.572 ns/op
```

#### 锁粗化

对相同对象多次加锁，导致线程发生多次重入，可以使用锁粗化方式来优化，这不同于之前讲的细分锁的粒度。 

### 3.7 wait&notify

#### 小故事 - 为什么需要 wait

- 由于条件不满足，小南不能继续进行计算

- 但小南如果一直占用着锁，其它人就得一直阻塞，效率太低

  ![为什么需要wait小故事1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546190.PNG)

- 于是老王单开了一间休息室（调用 wait 方法），让小南到休息室（WaitSet）等着去了，但这时锁释放开，其它人可以由老王随机安排进屋

- 直到小M将烟送来，大叫一声 [ 你的烟到了 ] （调用 notify 方法）

  ![为什么需要wait小故事2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546664.PNG)

- 小南于是可以离开休息室，重新进入竞争锁的队列

  ![为什么需要wait小故事3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546862.PNG)

#### 原理：wait / notify

![wait-notify](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546238.PNG)

- Owner 线程发现条件不满足，调用 wait 方法，即可进入 WaitSet 变为 WAITING 状态
- BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片
- BLOCKED 线程会在 Owner 线程释放锁时唤醒
- WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入EntryList 重新竞争 

#### API 介绍

- `obj.wait()` 让进入 object 监视器的线程到 waitSet 等待
- `obj.notify() `在 object 上正在 waitSet 等待的线程中挑一个唤醒
- `obj.notifyAll() `让 object 上正在 waitSet 等待的线程全部唤醒

它们都是线程之间进行协作的手段，都属于 Object 对象的方法。**必须获得此对象的锁，才能调用这几个方法**

```java
public class Test07 {

    final static Object lock = new Object();

    public static void main(String[] args) {
        new Thread(()->{
            synchronized (lock){
                LoggerUtils.LOGGER.debug("执行...");
                try {
                    lock.wait();  // 让线程在obj上一直等待下去
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                LoggerUtils.LOGGER.debug("其他代码...");
            }
        }, "t1").start();

        new Thread(()->{
            synchronized (lock){
                LoggerUtils.LOGGER.debug("执行...");
                try {
                    lock.wait(); // 让线程在obj上一直等待下去
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                LoggerUtils.LOGGER.debug("其它代码....");
            }
        }, "t2").start();

        // 主线程两秒后执行
        Sleeper.sleep(2);
        LoggerUtils.LOGGER.debug("唤醒 obj 上其它线程");
        synchronized (lock){
            lock.notify();  // 唤醒obj上一个线程
            // lock.notifyAll();   // 唤醒obj上所有等待线程
        }
    }
}
```

notify的一种结果：

```
10:23:38.931 cn.util.LoggerUtils [t1] - 执行...
10:23:38.934 cn.util.LoggerUtils [t2] - 执行...
10:23:40.679 cn.util.LoggerUtils [main] - 唤醒 obj 上其它线程
10:23:40.679 cn.util.LoggerUtils [t1] - 其他代码...
```

notifyAll 的结果：

```
10:24:06.208 cn.util.LoggerUtils [t1] - 执行...
10:24:06.210 cn.util.LoggerUtils [t2] - 执行...
10:24:07.931 cn.util.LoggerUtils [main] - 唤醒 obj 上其它线程
10:24:07.931 cn.util.LoggerUtils [t2] - 其它代码....
10:24:07.931 cn.util.LoggerUtils [t1] - 其他代码...
```

`wait()` 方法会释放对象的锁，进入 WaitSet 等待区，从而让其他线程有机会获取对象的锁。无限制等待，直到notify 为止

`wait(long n)` 有时限的等待, 到 n 毫秒后结束等待，或是被 notify

### 3.8 wait&notify 的正确姿势

开始之前先看看

#### sleep(long n) 和 wait(long n) 的区别

1. sleep 是 Thread 的静态方法，而 wait 是 Object 的方法；
2. sleep 不需要强制和 synchronized 配合使用，**但 wait 需要和 synchronized 一起用**；
3. sleep 在睡眠的同时，**不会释放对象锁的**，但 wait 在等待的时候会释放对象锁 
4. 共同点：它们状态 TIMED_WAITING

**Step1**

```java
static final Object room = new Object();
static boolean hasCigarette = false;
static boolean hasTakeout = false;
```

思考下面的解决方案好不好，为什么？ 

```java
static final Object room = new Object();
static boolean hasCigarette = false;
static boolean hasTakeout = false;

public static void main(String[] args) {

    new Thread(()->{
        synchronized (room){
            LoggerUtils.LOGGER.debug("有烟没？[{}]", hasCigarette);
            if(!hasCigarette){
                LoggerUtils.LOGGER.debug("没烟，先歇会！");
                Sleeper.sleep(2);
            }
            LoggerUtils.LOGGER.debug("有烟没？[{}]", hasCigarette);
            if(hasCigarette){
                LoggerUtils.LOGGER.debug("可以开始干活了");
            }
        }
    }, "小南").start();

    for (int i = 0; i < 5; i++) {
        new Thread(()->{
            synchronized (room){
                LoggerUtils.LOGGER.debug("其他人，可以开始干活了");
            }
        }, "其他人"+i).start();
    }

    Sleeper.sleep(1);
    new Thread(()->{
        hasCigarette = true;
        LoggerUtils.LOGGER.debug("烟到了噢！");
    }, "宋岩德").start();
}
```

输出：

```
10:30:37.504 cn.util.LoggerUtils [小南] - 有烟没？[false]
10:30:37.507 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:30:38.302 cn.util.LoggerUtils [宋岩德] - 烟到了噢！
10:30:39.508 cn.util.LoggerUtils [小南] - 有烟没？[true]
10:30:39.508 cn.util.LoggerUtils [小南] - 可以开始干活了
10:30:39.508 cn.util.LoggerUtils [其他人4] - 其他人，可以开始干活了
10:30:39.508 cn.util.LoggerUtils [其他人3] - 其他人，可以开始干活了
10:30:39.508 cn.util.LoggerUtils [其他人2] - 其他人，可以开始干活了
10:30:39.508 cn.util.LoggerUtils [其他人1] - 其他人，可以开始干活了
10:30:39.508 cn.util.LoggerUtils [其他人0] - 其他人，可以开始干活了

```

- 其它干活的线程，都要一直阻塞，效率太低
- 小南线程必须睡足 2s 后才能醒来，就算烟提前送到，也无法立刻醒来
- 加了 synchronized (room) 后，就好比小南在里面反锁了门睡觉，烟根本没法送进门，main 没加synchronized 就好像 main 线程是翻窗户进来的
- 解决方法，使用 wait - notify 机制

**step 2**

思考下面的实现行吗，为什么？

```java
static final Object room = new Object();
static boolean hasCigarette = false;
static boolean hasTakeout = false;

public static void main(String[] args) {

    new Thread(()->{
        synchronized (room){
            LoggerUtils.LOGGER.debug("有烟没？[{}]", hasCigarette);
            if(!hasCigarette){
                LoggerUtils.LOGGER.debug("没烟，先歇会！");
                try {
                    room.wait(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            LoggerUtils.LOGGER.debug("有烟没？[{}]", hasCigarette);
            if(hasCigarette){
                LoggerUtils.LOGGER.debug("可以开始干活了");
            }
        }
    }, "小南").start();

    for (int i = 0; i < 5; i++) {
        new Thread(()->{
            synchronized (room){
                LoggerUtils.LOGGER.debug("其他人，可以开始干活了");
            }
        }, "其他人"+i).start();
    }

    Sleeper.sleep(1);
    new Thread(()->{
        synchronized (room){
            hasCigarette = true;
            LoggerUtils.LOGGER.debug("烟到了噢！");
            room.notify();
        }
    }, "宋岩德").start();
}
```

输出：

```
10:32:58.719 cn.util.LoggerUtils [小南] - 有烟没？[false]
10:32:58.722 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:32:58.722 cn.util.LoggerUtils [其他人4] - 其他人，可以开始干活了
10:32:58.722 cn.util.LoggerUtils [其他人3] - 其他人，可以开始干活了
10:32:58.722 cn.util.LoggerUtils [其他人2] - 其他人，可以开始干活了
10:32:58.722 cn.util.LoggerUtils [其他人1] - 其他人，可以开始干活了
10:32:58.722 cn.util.LoggerUtils [其他人0] - 其他人，可以开始干活了
10:32:59.500 cn.util.LoggerUtils [宋岩德] - 烟到了噢！
10:32:59.500 cn.util.LoggerUtils [小南] - 有烟没？[true]
10:32:59.500 cn.util.LoggerUtils [小南] - 可以开始干活了
```

- 解决了其它干活的线程阻塞的问题
- 但如果有其它线程也在等待条件呢？

**Step3**

```java
public static void main(String[] args) {

    new Thread(()->{
        synchronized (room){
            LoggerUtils.LOGGER.debug("有烟没？[{}]", hasCigarette);
            if(!hasCigarette){
                LoggerUtils.LOGGER.debug("没烟，先歇会！");
                try {
                    room.wait(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            LoggerUtils.LOGGER.debug("有烟没？[{}]", hasCigarette);
            if(hasCigarette){
                LoggerUtils.LOGGER.debug("可以开始干活了");
            }else{
                LoggerUtils.LOGGER.debug("没干成活...");
            }
        }
    }, "小南").start();

    new Thread(() -> {
        synchronized (room) {
            LoggerUtils.LOGGER.debug("外卖送到没？[{}]", hasTakeout);
            if (!hasTakeout) {
                LoggerUtils.LOGGER.debug("没外卖，先歇会！");
                try {
                    room.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            LoggerUtils.LOGGER.debug("外卖送到没？[{}]", hasTakeout);
            if (hasTakeout) {
                LoggerUtils.LOGGER.debug("可以开始干活了");
            } else {
                LoggerUtils.LOGGER.debug("没干成活...");
            }
        }
    }, "小女").start();

    Sleeper.sleep(1);
    new Thread(()->{
        synchronized (room){
            hasTakeout = true;
            LoggerUtils.LOGGER.debug("外卖到了噢！");
            room.notify();
        }
    }, "外卖小哥").start();
}
```

输出：

```
10:36:37.367 cn.util.LoggerUtils [小南] - 有烟没？[false]
10:36:37.372 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:36:37.376 cn.util.LoggerUtils [小女] - 外卖送到没？[false]
10:36:37.376 cn.util.LoggerUtils [小女] - 没外卖，先歇会！
10:36:38.147 cn.util.LoggerUtils [外卖小哥] - 外卖到了噢！
10:36:38.149 cn.util.LoggerUtils [小南] - 有烟没？[false]
10:36:38.149 cn.util.LoggerUtils [小南] - 没干成活...
```

- notify 只能随机唤醒一个 WaitSet 中的线程，这时如果有其它线程也在等待，那么就可能唤醒不了正确的线程，称之为【虚假唤醒】
- 解决方法，改为 notifyAll

**step4**

```java
// 将上述代码中的
room.notify();
// 修改为
room.notifyAll();
```

输出：

```
10:37:43.968 cn.util.LoggerUtils [小南] - 有烟没？[false]
10:37:43.971 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:37:43.971 cn.util.LoggerUtils [小女] - 外卖送到没？[false]
10:37:43.972 cn.util.LoggerUtils [小女] - 没外卖，先歇会！
10:37:44.725 cn.util.LoggerUtils [外卖小哥] - 外卖到了噢！
10:37:44.725 cn.util.LoggerUtils [小女] - 外卖送到没？[true]
10:37:44.725 cn.util.LoggerUtils [小女] - 可以开始干活了
10:37:44.725 cn.util.LoggerUtils [小南] - 有烟没？[false]
10:37:44.725 cn.util.LoggerUtils [小南] - 没干成活...
```

- 用 notifyAll 仅解决某个线程的唤醒问题，但使用 if + wait 判断仅有一次机会，一旦条件不成立，就没有重新判断的机会了
- 解决方法，用 while + wait，当条件不成立，再次 wait

**step 5**

将 if 改为 while

```java
while (!hasCigarette){
    LoggerUtils.LOGGER.debug("没烟，先歇会！");
    try {
        room.wait(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

while (!hasTakeout) {
    LoggerUtils.LOGGER.debug("没外卖，先歇会！");
    try {
        room.wait();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

改动后输出：

```
10:39:43.580 cn.util.LoggerUtils [小南] - 有烟没？[false]
10:39:43.583 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:39:43.583 cn.util.LoggerUtils [小女] - 外卖送到没？[false]
10:39:43.583 cn.util.LoggerUtils [小女] - 没外卖，先歇会！
10:39:44.343 cn.util.LoggerUtils [外卖小哥] - 外卖到了噢！
10:39:44.343 cn.util.LoggerUtils [小女] - 外卖送到没？[true]
10:39:44.343 cn.util.LoggerUtils [小女] - 可以开始干活了
10:39:44.343 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:39:46.343 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:39:48.343 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:39:50.343 cn.util.LoggerUtils [小南] - 没烟，先歇会！
10:39:52.344 cn.util.LoggerUtils [小南] - 没烟，先歇会！
```

### 模式：保护性暂停

#### 定义

即 Guarded Suspension，用在**一个线程等待另一个线程的执行结果**

要点

- 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个 GuardedObject
- 如果有结果不断从一个线程到另一个线程那么可以使用消息队列（见生产者/消费者）
- JDK 中，join 的实现、Future 的实现，采用的就是此模式
- 因为要等待另一方的结果，因此归类到同步模式 

![同步模式之保护性暂停](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546493.PNG)

#### 实现

`class GuardedObject-->v1`

```java
private Object response;
private final Object lock = new Object();

public Object get(){
    synchronized (lock){
        // 条件不满足则等待
        while (response == null){
            try {
                lock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return response;
    }
}

public void complete(Object response){
    synchronized (lock){
        // 条件满足，通知等待线程
        this.response = response;
        lock.notifyAll();
    }
}
```

**应用**

一个线程等待另一个线程的执行结果

```java
public static void main(String[] args) {
    GuardedObject guardedObject = new GuardedObject();
    new Thread(()->{
        // 子线程执行下载
        Object response = new Object();
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("download complete...");
        guardedObject.complete(response);
    }, "download").start();

    LoggerUtils.LOGGER.debug("waiting...");
    // 主线程阻塞等待
    Object response = guardedObject.get();
    LoggerUtils.LOGGER.debug("get response: {}", response);
}
```

执行结果

```java
13:59:54.130 cn.util.LoggerUtils [main] - waiting...
13:59:54.873 cn.util.LoggerUtils [download] - download complete...
13:59:54.873 cn.util.LoggerUtils [main] - get response: java.lang.Object@725bef66
```

#### 带超时版 GuardedObject 

如果要控制超时时间呢：`class GuardedObject-->v2`

```java
private Object response;
private final Object lock = new Object();

public Object get(long mills){
    synchronized (lock){
        // 1) 记录最初时间
        long begin = System.currentTimeMillis();
        // 2) 已经经历的时间
        long timePassed = 0;
        // 条件不满足则等待
        while (response == null){
            long waitTime = mills - timePassed;
            LoggerUtils.LOGGER.debug("waitTime:{}", waitTime);
            if(waitTime <= 0){
                // 等待时间已经超过设定的时间，直接break
                LoggerUtils.LOGGER.debug("break...");
                break;
            }
            try {
                lock.wait(waitTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 3) 如果提前被唤醒，这时已经经历的时间假设为 400
            timePassed = System.currentTimeMillis() - begin;
            LoggerUtils.LOGGER.debug("timePassed:{}, Object is null:{}", timePassed, response==null);
        }
        return response;
    }
}

public void complete(Object response){
    synchronized (lock){
        // 条件满足，通知等待线程
        this.response = response;
        lock.notifyAll();
    }
}
```

测试，没有超时

```java
public static void main(String[] args) {
    GuardedObjectV2 guardedObject = new GuardedObjectV2();
    new Thread(()->{
        Sleeper.sleep(1);
        guardedObject.complete(null);
        Sleeper.sleep(1);
        guardedObject.complete(new Object());
    }, "download").start();

    Object response = guardedObject.get(2500);
    LoggerUtils.LOGGER.debug("get response: {}", response);
}
```

输出

```
14:17:10.031 cn.util.LoggerUtils [main] - waitTime:2500
14:17:10.788 cn.util.LoggerUtils [main] - timePassed:1006, Object is null:true
14:17:10.788 cn.util.LoggerUtils [main] - waitTime:1494
14:17:11.788 cn.util.LoggerUtils [main] - timePassed:2006, Object is null:false
14:17:11.788 cn.util.LoggerUtils [main] - get response: java.lang.Object@6e3c1e69
```

测试，超时

```java
Object response = guardedObject.get(1500);
```

输出

```
14:18:11.454 cn.util.LoggerUtils [main] - waitTime:1500
14:18:12.180 cn.util.LoggerUtils [main] - timePassed:1004, Object is null:true
14:18:12.180 cn.util.LoggerUtils [main] - waitTime:496
14:18:12.676 cn.util.LoggerUtils [main] - timePassed:1500, Object is null:true
14:18:12.676 cn.util.LoggerUtils [main] - waitTime:0
14:18:12.676 cn.util.LoggerUtils [main] - break...
14:18:12.676 cn.util.LoggerUtils [main] - get response: null
```

#### 原理：join

是调用者轮询检查线程 alive 状态

```java
t1.join();
```

等价于下面的代码

```java
synchronized (t1) {
    // 调用者线程进入 t1 的 waitSet 等待, 直到 t1 运行结束
    while (t1.isAlive()) {
        t1.wait(0);
    }
}

// Join源码
public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

#### 多任务版 GuardedObject 

图中 Futures 就好比居民楼一层的信箱（每个信箱有房间编号），左侧的 t0，t2，t4 就好比等待邮件的居民，右侧的 t1，t3，t5 就好比邮递员 

如果需要在多个类之间使用 GuardedObject 对象，作为参数传递不是很方便，因此设计一个用来解耦的中间类，这样不仅能够解耦【结果等待者】和【结果生产者】，还能够同时支持多个任务的管理 

![多任务GuardedObject](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546497.PNG)

新增 id 用来标识 Guarded Object：

```java
public class GuardedObjectV3 {

    // 标识 Guarded Object
    private int id;

    public GuardedObjectV3(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    // 结果
    private Object response;

    // 获取结果
    // timeout 表示要等待多久,如：2000
    public Object get(long timeout){
        synchronized (this){
            // 开始时间
            long begin = System.currentTimeMillis();
            // 经历的时间
            long passedTime = 0;
            while (response == null){
                // 这一轮循环应该等待的时间
                long waitTime = timeout - passedTime;
                // 经历的时间超过了最大等待时间时，退出循环
                if(waitTime <= 0){
                    break;
                }
                try {
                    this.wait(waitTime);  // 存在虚假唤醒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 求得经历时间
                passedTime = System.currentTimeMillis() - begin;
            }
            return response;
        }
    }

    // 产生结果
    public void complete(Object response){
        synchronized (this){
            // 给结果成员变量赋值
            this.response = response;
            this.notifyAll();
        }
    }
}
```

中间解耦类：

```java
class Mailboxes{

    // 这里采用线程安全的hashtable
    private static Map<Integer, GuardedObjectV3> boxes = new Hashtable<>();

    private static int id = 1;

    // 产生唯一 id
    private static synchronized int generateId(){
        return id++;
    }

    public static GuardedObjectV3 getGuardedObjectV3(int id){
        // 用完就删，为了防止堆溢出
        return boxes.remove(id);
    }

    public static GuardedObjectV3 createGuardedObjectV3(){
        GuardedObjectV3 go = new GuardedObjectV3(generateId());
        boxes.put(go.getId(), go);
        return go;
    }

    public static Set<Integer> getIds(){
        return boxes.keySet();
    }
}
```

业务相关类：

```java
class Person extends Thread{
    @Override
    public void run() {
        // 收信
        GuardedObjectV3 guardedObject = Mailboxes.createGuardedObjectV3();
        LoggerUtils.LOGGER.debug("开始收信 id:{}", guardedObject.getId());
        Object result = guardedObject.get(5000l);
        LoggerUtils.LOGGER.debug("收到信 id:{}, 内容:{}", guardedObject.getId(), result);
    }
}

class Postman extends Thread{
    private int id;
    private String mail;

    public Postman(int id, String mail){
        this.id = id;
        this.mail = mail;
    }

    @Override
    public void run() {
        GuardedObjectV3 guardedObject = Mailboxes.getGuardedObjectV3(id);
        LoggerUtils.LOGGER.debug("送信 id:{}, 内容:{}", id, mail);
        guardedObject.complete(mail);
    }
}
```

测试

```java
public static void main(String[] args) {
    for (int i = 0; i < 3; i++) {
        new Person().start();
    }
    Sleeper.sleep(1);

    for(Integer id: Mailboxes.getIds()){
        new Postman(id, "内容"+id).start();
    }
}
```

某次运行结果：

```java
14:52:35.038 cn.util.LoggerUtils [Thread-1] - 开始收信 id:2
14:52:35.040 cn.util.LoggerUtils [Thread-2] - 开始收信 id:3
14:52:35.041 cn.util.LoggerUtils [Thread-0] - 开始收信 id:1
14:52:35.802 cn.util.LoggerUtils [Thread-4] - 送信 id:2, 内容:内容2
14:52:35.802 cn.util.LoggerUtils [Thread-3] - 送信 id:3, 内容:内容3
14:52:35.802 cn.util.LoggerUtils [Thread-5] - 送信 id:1, 内容:内容1
14:52:35.802 cn.util.LoggerUtils [Thread-1] - 收到信 id:2, 内容:内容2
14:52:35.803 cn.util.LoggerUtils [Thread-2] - 收到信 id:3, 内容:内容3
14:52:35.803 cn.util.LoggerUtils [Thread-0] - 收到信 id:1, 内容:内容1
```

### 模式：生产者消费者

#### 定义 

要点

- 与前面的保护性暂停中的 GuardObject 不同，不需要产生结果和消费结果的线程一一对应
- 消费队列可以用来**平衡生产和消费的线程资源**
- 生产者**仅负责**产生结果数据，不关心数据该如何处理，而消费者**专心处理**结果数据
- 消息队列是有容量限制的，满时不会再加入数据，空时不会再消耗数据
- JDK 中各种阻塞队列，采用的就是这种模式 

![生产者消费者](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546919.PNG)

#### 实现

```java
class Message{
    private int id;
    private Object message;

    public Message(int id, Object message) {
        this.id = id;
        this.message = message;
    }

    public int getId() {
        return id;
    }

    public Object getMessage() {
        return message;
    }

    @Override
    public String toString() {
        return "Message{" +
                "id=" + id +
                '}';
    }
}

class MessageQueue{

    private LinkedList<Message> queue;
    private int capacity;

    public MessageQueue(int capacity) {
        this.capacity = capacity;
        this.queue = new LinkedList<>();
    }

    public Message take(){
        synchronized (queue){
            while (queue.isEmpty()){
                LoggerUtils.LOGGER.debug("没货了, wait");
                try {
                    queue.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            Message message = queue.removeFirst();
            queue.notifyAll();
            return message;
        }
    }

    public void put(Message message){
        synchronized (queue){
            while (queue.size() == capacity){
                LoggerUtils.LOGGER.debug("库存已达上限, wait");
                try {
                    queue.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            
            queue.addLast(message);
            queue.notifyAll();
        }
    }
}
```

**应用**

```java
public class ConsumerAndProducer {

    public static void main(String[] args) {

        MessageQueue queue = new MessageQueue(2);

        // 4 个生产者线程, 下载任务
        for (int i = 0; i < 4; i++) {
            int id = i;
            new Thread(()->{
                LoggerUtils.LOGGER.debug("create producer..." + id);
                Message message = new Message(id, new Object());
                queue.put(message);
            }, "生产者"+i).start();
        }

        // 1 个消费者线程, 处理结果
        new Thread(()->{
            while (true){
                Message message = queue.take();
                LoggerUtils.LOGGER.debug("consumer:"+message);
            }
        }, "消费者").start();
    }
}
```

某次运行结果：

```java
22:54:15.900 cn.util.LoggerUtils [生产者1] - create producer...1
22:54:15.900 cn.util.LoggerUtils [消费者] - 没货了, wait
22:54:15.900 cn.util.LoggerUtils [生产者2] - create producer...2
22:54:15.900 cn.util.LoggerUtils [生产者3] - create producer...3
22:54:15.900 cn.util.LoggerUtils [生产者0] - create producer...0
22:54:15.903 cn.util.LoggerUtils [生产者3] - 库存已达上限, wait
22:54:15.903 cn.util.LoggerUtils [生产者2] - 库存已达上限, wait
22:54:15.903 cn.util.LoggerUtils [生产者3] - 库存已达上限, wait
22:54:15.903 cn.util.LoggerUtils [消费者] - consumer:Message{id=1}
22:54:15.903 cn.util.LoggerUtils [消费者] - consumer:Message{id=0}
22:54:15.903 cn.util.LoggerUtils [消费者] - consumer:Message{id=2}
22:54:15.903 cn.util.LoggerUtils [消费者] - consumer:Message{id=3}
22:54:15.903 cn.util.LoggerUtils [消费者] - 没货了, wait
```

### 3.9 Park & Unpark

基本使用

它们是 LockSupport 类中的方法

```java
// 暂停当前线程
LockSupport.park();
// 恢复某个线程的运行
LockSupport.unpark(暂停线程对象);
```

先park再unpark

```java
public static void main(String[] args) {
    Thread t1 = new Thread(()->{
        LoggerUtils.LOGGER.debug("start...");
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("park...");
        LockSupport.park();
        LoggerUtils.LOGGER.debug("resume...");
    }, "t1");
    t1.start();

    Sleeper.sleep(3);
    LoggerUtils.LOGGER.debug("unpark...");
    LockSupport.unpark(t1);
}
```

输出：

```
19:09:50.598 cn.util.LoggerUtils [t1] - start...
19:09:51.600 cn.util.LoggerUtils [t1] - park...
19:09:53.388 cn.util.LoggerUtils [main] - unpark...
19:09:53.388 cn.util.LoggerUtils [t1] - resume...
```

先unpark再park

```java
public static void main(String[] args) {
    Thread t1 = new Thread(()->{
        LoggerUtils.LOGGER.debug("start...");
        Sleeper.sleep(2);
        LoggerUtils.LOGGER.debug("park...");
        LockSupport.park();
        LoggerUtils.LOGGER.debug("resume...");
    }, "t1");
    t1.start();

    Sleeper.sleep(1);
    LoggerUtils.LOGGER.debug("unpark...");
    LockSupport.unpark(t1);
}
```

输出：

```
19:11:13.297 cn.util.LoggerUtils [t1] - start...
19:11:14.073 cn.util.LoggerUtils [main] - unpark...
19:11:15.299 cn.util.LoggerUtils [t1] - park...
19:11:15.299 cn.util.LoggerUtils [t1] - resume...
```

#### 特点

与 Object 的 wait & notify 相比

- wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必
- park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll是唤醒所有等待线程，就不那么【精确】
- park & unpark 可以先 unpark，而 wait & notify 不能先 notify

#### 原理：park & unpark

每个线程都有自己的一个 Parker 对象，由三部分组成 `_counter` ， `_cond` 和 `_mutex` 

打个比喻：

- 线程就像一个旅人，Parker 就像他随身携带的背包，条件变量就好比背包中的帐篷。`_counter` 就好比背包中的备用干粮（0 为耗尽，1 为充足）
- 调用 park 就是要看需不需要停下来歇息
  - 如果备用干粮耗尽，那么钻进帐篷歇息
  - 如果备用干粮充足，那么不需停留，继续前进
- 调用 unpark，就好比令干粮充足
  - 如果这时线程还在帐篷，就唤醒让他继续前进
  - 如果这时线程还在运行，那么下次他调用 park 时，仅是消耗掉备用干粮，不需停留继续前进
  - 因为背包空间有限，多次调用 unpark 仅会补充一份备用干粮 

![park1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546409.PNG)

1. 当前线程调用 Unsafe.park() 方法
2. 检查 _counter ，本情况为 0，这时，获得 _mutex 互斥锁
3. 线程进入 _cond 条件变量阻塞
4. 设置 _counter = 0 

![park2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546812.PNG)

1. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为 1
2. 唤醒 _cond 条件变量中的 Thread_0
3. Thread_0 恢复运行
4. 设置 _counter 为 0 

![park3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251546549.PNG)

1. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为 1
2. 当前线程调用 Unsafe.park() 方法
3. 检查 _counter ，本情况为 1，这时线程无需阻塞，继续运行
4. 设置 _counter 为 0 