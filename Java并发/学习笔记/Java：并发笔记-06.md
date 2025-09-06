# Java：并发笔记-06

> 说明：观看了 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444/)** 的课程 [java并发编程](https://www.bilibili.com/video/BV16J411h7Rd) 后所作的学习笔记，总计拆分为9篇笔记，此为6/9。

## 5. 共享模型之无锁 

### 本章内容

- CAS 与 volatile
- 原子整数
- 原子引用
- 原子累加器
- Unsafe 

### 5.1 问题提出 

有如下需求，保证 `account.withdraw` 取款方法的线程安全 

```java
interface Account{
    // 获取余额
    Integer getBalance();

    // 取款
    void withdraw(Integer amount);

    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元的操作
     * 如果初始余额为10000那么正确的结果应当是 0
     */
    static void demo(Account account){
        List<Thread> threads = new ArrayList<>();
        long start = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            threads.add(new Thread(()->{
                account.withdraw(10);
            }));
        }

        threads.forEach(Thread::start);
        threads.forEach(t->{
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end = System.nanoTime();
        System.out.println(account.getBalance()
                + " cost: " + (end-start)/1000_000 + " ms");
    }
}
```

原有实现并不是线程安全的：

```java
class AccountUnsafe implements Account{
    
    private Integer balance;

    public AccountUnsafe(Integer balance) {
        this.balance = balance;
    }

    @Override
    public Integer getBalance() {
        return balance;
    }

    @Override
    public void withdraw(Integer amount) {
        balance -= amount;
    }
}
```

执行测试代码：

```java
public static void main(String[] args) {
    Account.demo(new AccountUnsafe(10000));
}
```

某次的执行结果：

```
350 cost: 132 ms
```

#### 为什么不安全

withdraw 方法：

```java
public void withdraw(Integer amount) {
    balance -= amount;
}
```

上述方法不具有原子性，因此会出现：

- 单核的指令交错
- 多核的指令交错 

#### 解决思路-锁

首先想到的是给 Account 对象加锁

```java
@Override
public synchronized Integer getBalance() {
    return balance;
}

@Override
public synchronized void withdraw(Integer amount) {
    balance -= amount;
}
```

结果为：

```
0 cost: 150 ms
```

#### 解决思路-无锁 

```java
class AccountSafe implements Account{

    private AtomicInteger balance;

    public AccountSafe(AtomicInteger balance) {
        this.balance = balance;
    }

    @Override
    public Integer getBalance() {
        return balance.get();
    }

    @Override
    public void withdraw(Integer amount) {
        while (true){
            int prev = balance.get();
            int next = prev - amount;
            if(balance.compareAndSet(prev, next)){
                break;
            }
        }

        // 可以简化为下面的方法
        // balance.addAndGet(-1 * amount);
    }
}
```

执行测试代码

```java
public static void main(String[] args) {
    Account.demo(new AccountSafe(10000));
}
```

某次的执行结果：

```
0 cost: 126 ms
```

### 5.2 CAS 与 volatile 

前面看到的 `AtomicInteger` 的解决方法，内部并没有用锁来保护共享变量的线程安全。那么它是如何实现的呢？ 

```java
@Override
public void withdraw(Integer amount) {
    while (true){
        int prev = balance.get();
        int next = prev - amount;

        /**
		  * compareAndSet 正是做这个检查，在 set 前，先比较 prev 与当前值
                - 不一致了，next 作废，返回 false 表示失败
                    比如，别的线程已经做了减法，当前值已经被减成了 990
                    那么本线程的这次 990 就作废了，进入 while 下次循环重试
                - 一致，以 next 设置为新值，返回 true 表示成功
		*/
        if(balance.compareAndSet(prev, next)){
            break;
        }
    }
    // 可以简化为下面的方法
    // balance.addAndGet(-1 * amount);
}
```

其中的关键是 `compareAndSet`，它的简称就是 CAS （也有 Compare And Swap 的说法），它必须是原子操作。 

![CAS](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548390.PNG)

> **注意** 
>
> 其实 CAS 的底层是 lock cmpxchg 指令（X86 架构），在单核 CPU 和多核 CPU 下都能够保证【比较-交换】的原子性。 
>
> 在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子的。 

#### 慢动作分析 

```java
public class SlowMotion {

    public static void main(String[] args) {
        AtomicInteger balance = new AtomicInteger(10000);
        int mainPrev = balance.get();
        LoggerUtils.LOGGER.debug("try get {}", mainPrev);

        new Thread(()->{
            Sleeper.sleep(1);
            int prev = balance.get();
            balance.compareAndSet(prev, 9000);
            LoggerUtils.LOGGER.debug(balance.toString());
        }, "t1").start();

        Sleeper.sleep(2);
        LoggerUtils.LOGGER.debug("try set 8000...");
        // false->在t1中已经把值修改了，即此时的原值已经不是10000了
        boolean isSuccess = balance.compareAndSet(mainPrev, 8000);
        LoggerUtils.LOGGER.debug("is success? {}", isSuccess);
        if(!isSuccess){
            mainPrev = balance.get();
            LoggerUtils.LOGGER.debug("try set 8000...");
            isSuccess = balance.compareAndSet(mainPrev, 8000);
            LoggerUtils.LOGGER.debug("is success? {}", isSuccess);
        }
    }
}
```

输出结果：

```java
13:06:31.015 cn.util.LoggerUtils [main] - try get 10000
13:06:32.054 cn.util.LoggerUtils [t1] - 9000
13:06:33.053 cn.util.LoggerUtils [main] - try set 8000...
13:06:33.053 cn.util.LoggerUtils [main] - is success? false
13:06:33.053 cn.util.LoggerUtils [main] - try set 8000...
13:06:33.053 cn.util.LoggerUtils [main] - is success? true
```

#### volatile 

获取共享变量时，为了保证该变量的可见性，需要使用 volatile 修饰。 

它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存。即一个线程对 volatile 变量的修改，对另一个线程可见。

> **注意**：volatile 仅仅保证了共享变量的可见性，让其它线程能够看到最新值，但不能解决指令交错问题（不能保证原子性）

CAS 必须借助 volatile 才能读取到共享变量的最新值来实现【比较并交换】的效果 

```java
public class AtomicInteger extends Number implements java.io.Serializable {
	private volatile int value;   
}
```

#### 为什么无锁效率高

- 无锁情况下，即使重试失败，线程始终在高速运行，没有停歇，而 synchronized 会让线程在没有获得锁的时候，发生上下文切换，进入阻塞。打个比喻：线程就好像高速跑道上的赛车，高速运行时，速度超快，一旦发生上下文切换，就好比赛车要减速、熄火，等被唤醒又得重新打火、启动、加速... 恢复到高速运行，代价比较大；
- 但无锁情况下，因为线程要保持运行，需要额外 CPU 的支持，CPU 在这里就好比高速跑道，没有额外的跑道，线程想高速运行也无从谈起，虽然不会进入阻塞，但由于没有分到时间片，仍然会进入可运行状态，还是会导致上下文切换。 

![线程的6种状态](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548378.PNG)

#### CAS 的特点

结合 CAS 和 volatile 可以实现无锁并发，适用于线程数少、多核 CPU 的场景下。 

- CAS 是基于**乐观锁**的思想：最乐观的估计，不怕别的线程来修改共享变量，就算改了也没关系，我吃亏点再重试呗。
- synchronized 是基于**悲观锁**的思想：最悲观的估计，得防着其它线程来修改共享变量，我上了锁你们都别想改，我改完了解开锁，你们才有机会。
- CAS 体现的是**无锁并发、无阻塞并发**，请仔细体会这两句话的意思
  - 因为没有使用 synchronized，所以线程不会陷入阻塞，这是效率提升的因素之一
  - 但如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响 

### 5.3 原子整数

J.U.C 并发包提供了：

- AtomicBoolean
- AtomicInteger
- AtomicLong 

以 AtomicInteger 为例：

```java
AtomicInteger i = new AtomicInteger(0);

// 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++
i.getAndIncrement();
// i = 1, 结果 i = 2, 返回 2），类似于 ++i
i.incrementAndGet();
// 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i
i.decrementAndGet();
// 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
i.getAndDecrement();

// 获取并加值（i = 0, 结果 i = 5, 返回 0）
i.getAndAdd(5);
// 加值并获取（i = 5, 结果 i = 0, 返回 0）
i.addAndGet(-5);

// 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
i.getAndUpdate(p -> p-2);
// 等价于：
i.getAndUpdate(new IntUnaryOperator() {
    @Override
    public int applyAsInt(int operand) {
        return operand-2;
    }
});

// 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
i.updateAndGet(p -> p + 2);

// 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
// getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的
// getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 final
i.getAndAccumulate(10, (p, x) -> p + x);
// 等价于：
i.getAndAccumulate(10, new IntBinaryOperator() {
    @Override
    public int applyAsInt(int left, int right) {
        return left+right;
    }
});

// 计算并获取（i = 10, p 为 i 的当前值, x 为参数1, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
i.accumulateAndGet(-10, (p, x) -> p + x);
```

### 5.4 原子引用

为什么需要原子引用类型？

- AtomicReference
- AtomicMarkableReference
- AtomicStampedReference 

有如下方法

```java
interface DecimalAccount{

    // 获取金额
    BigDecimal getBalance();

    // 取款
    void withdraw(BigDecimal amount);

    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void demo(DecimalAccount account){
        long start = System.nanoTime();
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            threads.add(new Thread(()->{
                account.withdraw(BigDecimal.TEN);
            }));
        }
        
        threads.forEach(Thread::start);
        threads.forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end = System.nanoTime();
        System.out.println(account.getBalance()
                + " cost: " + (end-start)/1000_000 + " ms");
    }
}
```

试着提供不同的 DecimalAccount 实现，实现安全的取款操作 

#### 不安全实现 

```java
class DecimalAccountUnsafe implements DecimalAccount{

    private BigDecimal balance;

    public DecimalAccountUnsafe(BigDecimal balance) {
        this.balance = balance;
    }

    @Override
    public BigDecimal getBalance() {
        return balance;
    }

    @Override
    public void withdraw(BigDecimal amount) {
        this.balance = this.getBalance().subtract(amount);
    }
}
```

#### 安全实现-使用锁

```java
class DecimalAccountSafeLock implements DecimalAccount{

    private final Object lock = new Object();
    private BigDecimal balance;

    public DecimalAccountSafeLock(BigDecimal balance) {
        this.balance = balance;
    }

    @Override
    public BigDecimal getBalance() {
        return balance;
    }

    @Override
    public void withdraw(BigDecimal amount) {
        synchronized (lock){
            this.balance = this.getBalance().subtract(amount);
        }
    }
}
```

#### 安全实现-使用 CAS

```java
class DecimalAccountSafeCas implements DecimalAccount{

    private AtomicReference<BigDecimal> ref;

    public DecimalAccountSafeCas(BigDecimal balance) {
        ref = new AtomicReference<>(balance);
    }

    @Override
    public BigDecimal getBalance() {
        return ref.get();
    }

    @Override
    public void withdraw(BigDecimal amount) {
        while (true){
            BigDecimal prev = ref.get();
            BigDecimal next = prev.subtract(amount);
            if(ref.compareAndSet(prev, next)){
                break;
            }
        }
    }
}
```

测试代码：

```java
public static void main(String[] args) {
    DecimalAccount.demo(new DecimalAccountUnsafe(new BigDecimal("10000")));
    DecimalAccount.demo(new DecimalAccountSafeLock(new BigDecimal("10000")));
    DecimalAccount.demo(new DecimalAccountSafeCas(new BigDecimal("10000")));
}
```

运行结果：

```
3040 cost: 142 ms
0 cost: 111 ms
0 cost: 81 ms
```

#### ABA 问题及解决 

**ABA 问题** 

```java
static AtomicReference<String> ref = new AtomicReference<>("A");

public static void main(String[] args) {

    LoggerUtils.LOGGER.debug("main start...");
    // 获取值 A
    // 这个共享变量被它线程修改过？
    String prev = ref.get();
    Other();
    Sleeper.sleep(1);
    // 尝试改为 C
    LoggerUtils.LOGGER.debug("change A->C {}", ref.compareAndSet(prev, "C"));
}

private static void Other(){
    new Thread(()->{
        LoggerUtils.LOGGER.debug("change A->B {}",
                                 ref.compareAndSet(ref.get(), "B"));
    }, "t1").start();

    Sleeper.sleep(0.5);

    new Thread(()->{
        LoggerUtils.LOGGER.debug("change B->A {}",
                                 ref.compareAndSet(ref.get(), "A"));
    }, "t2").start();
}
```

输出：

```java
14:12:56.397 cn.util.LoggerUtils [main] - main start...
14:12:56.443 cn.util.LoggerUtils [t1] - change A->B true
14:12:56.949 cn.util.LoggerUtils [t2] - change B->A true
14:12:57.944 cn.util.LoggerUtils [main] - change A->C true
```

主线程仅能判断出共享变量的值与最初值 A 是否相同，不能感知到这种从 A 改为 B 又改回 A 的情况，如果主线程希望： 只要有其它线程【动过了】共享变量，那么自己的 cas 就算失败，这时，仅比较值是不够的，需要再加一个版本号

#### AtomicStampedReference

```java
static AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);

public static void main(String[] args) {

    LoggerUtils.LOGGER.debug("main start...");
    // 获取值 A
    String prev = ref.getReference();
    // 获取版本号
    int stamp = ref.getStamp();
    LoggerUtils.LOGGER.debug("value:{} stamp:{}", prev, stamp);

    Other();
    Sleeper.sleep(1);
    // 尝试改为 C
    LoggerUtils.LOGGER.debug("change A->C {}", ref.compareAndSet(prev, "C", stamp, stamp+1));
}

private static void Other(){
    new Thread(()->{
        LoggerUtils.LOGGER.debug("change A->B {}",
                                 ref.compareAndSet(ref.getReference(), "B", ref.getStamp(), ref.getStamp()+1));
        LoggerUtils.LOGGER.debug("更新版本为 {}",ref.getStamp());
    }, "t1").start();

    Sleeper.sleep(0.5);

    new Thread(()->{
        LoggerUtils.LOGGER.debug("change B->A {}",
                                 ref.compareAndSet(ref.getReference(), "A", ref.getStamp(), ref.getStamp()+1));
        LoggerUtils.LOGGER.debug("更新版本为 {}",ref.getStamp());
    }, "t2").start();
}
```

输出为：

```
14:20:39.699 cn.util.LoggerUtils [main] - main start...
14:20:39.704 cn.util.LoggerUtils [main] - value:A stamp:0
14:20:39.746 cn.util.LoggerUtils [t1] - change A->B true
14:20:39.747 cn.util.LoggerUtils [t1] - 更新版本为 1
14:20:40.256 cn.util.LoggerUtils [t2] - change B->A true
14:20:40.256 cn.util.LoggerUtils [t2] - 更新版本为 2
14:20:41.250 cn.util.LoggerUtils [main] - change A->C false
```

#### AtomicMarkableReference 

`AtomicStampedReference` 可以给原子引用加上版本号，追踪原子引用整个的变化过程，如： A -> B -> A ->C ，通过`AtomicStampedReference`，我们可以知道，引用变量中途被更改了几次。但是有时候，并不关心引用变量更改了几次，只是单纯的关心**是否更改过**，所以就有了`AtomicMarkableReference` 

![AtomicMarkableReference举例](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548735.PNG)

```java
public class TestABAAtomicMarkableReference {

    public static void main(String[] args) throws InterruptedException {
        GarbageBag bag = new GarbageBag("装满了垃圾");
        // 参数2 mark 可以看作一个标记，表示垃圾袋满了
        AtomicMarkableReference<GarbageBag> ref = new AtomicMarkableReference<>(bag, true);

        LoggerUtils.LOGGER.debug("主线程 start...");
        GarbageBag prev = ref.getReference();
        LoggerUtils.LOGGER.debug(prev.toString());

        new Thread(()->{
            LoggerUtils.LOGGER.debug("保洁阿姨线程 start...");
            bag.setDesc("空垃圾袋");
            while (!ref.compareAndSet(prev, bag, true, false)){

            }
            LoggerUtils.LOGGER.debug(bag.toString());
        }, "t1").start();

        Thread.sleep(1000);
        LoggerUtils.LOGGER.debug("主线程想要换一个垃圾袋...");
        boolean isSuccess = ref.compareAndSet(prev, new GarbageBag("空垃圾袋"), true, false);
        LoggerUtils.LOGGER.debug("换成功了吗？{}", isSuccess);
        LoggerUtils.LOGGER.debug(ref.getReference().toString());
    }

}

class GarbageBag{
    String desc;

    public GarbageBag(String desc) {
        this.desc = desc;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }

    @Override
    public String toString() {
        return "GarbageBag{" +
                "desc='" + desc + '\'' +
                '}';
    }
}
```

输出：

```java
14:32:00.700 cn.util.LoggerUtils [main] - 主线程 start...
14:32:00.703 cn.util.LoggerUtils [main] - GarbageBag{desc='装满了垃圾'}
14:32:00.745 cn.util.LoggerUtils [t1] - 保洁阿姨线程 start...
14:32:00.745 cn.util.LoggerUtils [t1] - GarbageBag{desc='空垃圾袋'}
14:32:01.746 cn.util.LoggerUtils [main] - 主线程想要换一个垃圾袋...
14:32:01.746 cn.util.LoggerUtils [main] - 换成功了吗？false
14:32:01.760 cn.util.LoggerUtils [main] - GarbageBag{desc='空垃圾袋'}
```

### 5.5 原子数组 

- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray 

有如下方法

```java
/**
 参数1，提供数组、可以是线程不安全数组或线程安全数组
 参数2，获取数组长度的方法
 参数3，自增方法，回传 array, index
 参数4，打印数组的方法
 */
// supplier 提供者 无中生有 ()->结果
// function 函数 一个参数一个结果 (参数)->结果 , BiFunction (参数1,参数2)->结果
// consumer 消费者 一个参数没结果 (参数)->void, BiConsumer (参数1,参数2)->
private static <T> void demo(
    Supplier<T> arraySupplier,
    Function<T, Integer> lengthFun,
    BiConsumer<T, Integer> putConsumer,
    Consumer<T> printConsumer){
    
    List<Thread> threads = new ArrayList<>();
    T array = arraySupplier.get();
    int length = lengthFun.apply(array);
    for (int i = 0; i < length; i++) {
        // 每个线程对数组作 10000 次操作,对每个索引下标都加上1000
        threads.add(new Thread(()->{
            for (int j = 0; j < 10000; j++) {
                putConsumer.accept(array, j%length);
            }
        }));
    }

    // 启动所有线程
    threads.forEach(Thread::start);
    // 等所有线程结束
    threads.forEach(thread -> {
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    printConsumer.accept(array);
}
```

#### 不安全的数组

```java
demo(
    ()->new int[10],
    (array)->array.length,
    (array, index)->array[index]++,
    array-> System.out.println(Arrays.toString(array))
);
```

结果：

```
[8930, 8930, 8939, 8944, 8949, 8952, 8922, 8930, 8924, 8932]
```

#### 安全的数组

```java
demo(
    ()-> new AtomicIntegerArray(10),  // 创建原子数组
    (array)->array.length(),
    (array, index)->array.getAndIncrement(index),
    array-> System.out.println(array)
);
```

结果：

```
[10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000]
```

### 5.6 字段更新器 

- AtomicReferenceFieldUpdater  // 引用类型  域字段
- AtomicIntegerFieldUpdater
- AtomicLongFieldUpdater 

利用字段更新器，可以针对对象的某个域（Field）进行原子操作，只能配合 volatile 修饰的字段使用，否则会出现异常 

```java
private int field;
Exception in thread "main" java.lang.IllegalArgumentException: Must be volatile type
```

```java
public class Test07 {

    private volatile int field;

    public static void main(String[] args) {

        AtomicIntegerFieldUpdater fieldUpdater =
                AtomicIntegerFieldUpdater.newUpdater(Test07.class, "field");

        Test07 test07 = new Test07();
        fieldUpdater.compareAndSet(test07, 0, 10);
        // 修改成功 field = 10
        System.out.println(test07.field);
        // 修改成功 field = 20
        fieldUpdater.compareAndSet(test07, 10, 20);
        System.out.println(test07.field);
        // 修改失败 field = 20
        fieldUpdater.compareAndSet(test07, 10, 30);
        System.out.println(test07.field);
    }
}
```

输出

```
10
20
20
```

### 5.7 原子累加器

- AtomicLong 
- LongAdder 

**累加器性能比较**

```java
private static <T> void demo(Supplier<T> adderSupplier, Consumer<T> action){
    
    T adder = adderSupplier.get();

    long start = System.nanoTime();
    List<Thread> threads = new ArrayList<>();
    // 40 个线程，每人累加 50 万
    for (int i = 0; i < 40; i++) {
        threads.add(new Thread(()->{
            for (int j = 0; j < 500000; j++) {
                action.accept(adder);
            }
        }));
    }

    threads.forEach(Thread::start);
    threads.forEach(thread -> {
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });

    long end = System.nanoTime();
    System.out.println(adder + " cost:" + (end - start)/1000_000 + "ms");
}
```

比较 AtomicLong 与 LongAdder 

```java
// 比较 LongAdder 与 AtomicLong
for (int i = 0; i < 5; i++) {
    demo(
        ()->new LongAdder(),
        (adder) -> adder.increment()
    );
}

for (int i = 0; i < 5; i++) {
    demo(
        ()->new AtomicLong(),
        (adder)->adder.incrementAndGet()
    );
}
```

输出：

```
20000000 cost:82ms
20000000 cost:94ms
20000000 cost:75ms
20000000 cost:63ms
20000000 cost:49ms
20000000 cost:487ms
20000000 cost:426ms
20000000 cost:419ms
20000000 cost:491ms
20000000 cost:460ms
```

性能提升的原因很简单，**LongAdder 就是在有竞争时，设置多个累加单元**，Therad-0 累加 Cell[0]，而 Thread-1 累加Cell[1]... 最后将结果汇总。这样它们在累加时操作的不同的 Cell 变量，因此减少了 CAS 重试失败，从而提高性能。 

#### 源码之LongAdder

LongAdder 是并发大师 @author Doug Lea （大哥李）的作品，设计的非常精巧

LongAdder 类有几个关键域 

```java
// 累加单元数组, 懒惰初始化
transient volatile Cell[] cells;

// 基础值, 如果没有竞争, 则用 cas 累加这个域
transient volatile long base;

// 在 cells 创建或扩容时, 置为1, 表示加锁
transient volatile int cellsBusy;
```

**cas 锁**

```java
// 不要用于实践！！！ 你不是大哥李
class LockCas{
    private AtomicInteger state = new AtomicInteger(0);

    public void lock(){
        while (true){
            if(state.compareAndSet(0, 1)){
                break;
            }
        }
    }

    public void unlock(){
        LoggerUtils.LOGGER.debug("unlock...");
        state.set(0);
    }
}
```

测试

```java
LockCas lock = new LockCas();
new Thread(()->{
    LoggerUtils.LOGGER.debug("begin...");
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("locking...");
        Sleeper.sleep(1);
    }finally {
        lock.unlock();
    }
}, "t1").start();

new Thread(()->{
    LoggerUtils.LOGGER.debug("begin...");
    lock.lock();
    try {
        LoggerUtils.LOGGER.debug("locking...");
    }finally {
        lock.unlock();
    }
}, "t2").start();
```

输出：

```java
19:18:54.696 cn.util.LoggerUtils [t1] - begin...
19:18:54.696 cn.util.LoggerUtils [t2] - begin...
19:18:54.698 cn.util.LoggerUtils [t1] - locking...
19:18:55.700 cn.util.LoggerUtils [t1] - unlock...
19:18:55.700 cn.util.LoggerUtils [t2] - locking...
19:18:55.700 cn.util.LoggerUtils [t2] - unlock...
```

#### 原理：伪共享

> 其中 Cell 即为累加单元，若一个缓存行中加入多个 Cell 单元，会造成伪共享。
>
> ```java
> // 防止 缓存行 伪共享
> @sun.misc.Contended
> static final class Cell {
> 	volatile long value;
> 	Cell(long x) { value = x; }
> 	
> 	// 最重要的方法, 用来 cas 方式进行累加, prev 表示旧值, next 表示新值
>     final boolean cas(long prev, long next) {
>     	return UNSAFE.compareAndSwapLong(this, valueOffset, prev, next);
>     }
> 	// 省略不重要代码
> }
> ```
>
> 得从缓存说起
>
> ![CPU内存缓存图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548810.PNG)
>
> 缓存与内存的速度比较 
>
> | 从 cpu 到 | 大约需要的时钟周期               |
> | --------- | -------------------------------- |
> | 寄存器    | 1 cycle (4GHz 的 CPU 约为0.25ns) |
> | L1        | 3~4 cycle                        |
> | L2        | 10~20 cycle                      |
> | L3        | 40~45 cycle                      |
> | 内存      | 120~240 cycle                    |
>
> 因为 CPU 与 内存的速度差异很大，需要靠预读数据至缓存来提升效率。
>
> 而缓存以**缓存行**为单位，每个缓存行对应着一块内存，一般是 64 byte（8 个 long）
>
> 缓存的加入会造成数据副本的产生，即同一份数据会缓存在不同核心的缓存行中
>
> **CPU 要保证数据的一致性，如果某个 CPU 核心更改了数据，其它 CPU 核心对应的整个缓存行必须失效** 
>
> ![累加单元被多个cpu核心使用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548610.PNG)
>
> 因为 Cell 是数组形式，在内存中是连续存储的，一个 Cell 为 24 字节（16 字节的对象头和 8 字节的 value），因此缓存行可以存下 2 个的 Cell 对象。这样问题来了： 
>
> - Core-0 要修改 Cell[0]
> - Core-1 要修改 Cell[1] 
>
> 无论谁修改成功，都会导致对方 Core 的缓存行失效，比如 Core-0 中 Cell[0]=6000, Cell[1]=8000 要累加Cell[0]=6001, Cell[1]=8000 ，这时会让 Core-1 的缓存行失效。
>
> `@sun.misc.Contended` 用来解决这个问题，它的原理是在使用此注解的对象或字段的前后各增加 128 字节大小的 padding，从而让 CPU 将对象预读至缓存时**占用不同的缓存行**，这样，不会造成对方缓存行的失效。
>
> ![解决CPU缓存行失效](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548551.PNG)
>
> 累加主要调用下面的方法
>
> ```java
> public void add(long x) {
>     // as 为累加单元数组
>     // b 为基础值
>     // x 为累加值
>     Cell[] as; long b, v; int m; Cell a;
>     // 进入 if 的两个条件
>     // 1. as 有值, 表示已经发生过竞争, 进入 if 
>     // 2. cas 给 base 累加时失败了, 表示 base 发生了竞争, 进入 if
>     if ((as = cells) != null || !casBase(b = base, b + x)) {
>         boolean uncontended = true;
>         // as 还没有创建
>         if (as == null || (m = as.length - 1) < 0 ||
>             // 当前线程对应的 cell 还没有
>             (a = as[getProbe() & m]) == null ||
>             // cas 给当前线程的 cell 累加失败 uncontended=false ( a 为当前线程的 cell )
>             !(uncontended = a.cas(v = a.value, v + x)))
>             // 进入 cell 数组创建、cell 创建的流程
>             longAccumulate(x, null, uncontended);
>     }
> }
> ```
>
> add 流程图
>
> ![LongAdder-add](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548152.PNG)
>
> ```java
> final void longAccumulate(long x, LongBinaryOperator fn,
>                           boolean wasUncontended) {
>     int h;
>     // 当前线程还没有对应的 cell, 需要随机生成一个 h 值用来将当前线程绑定到 cell
>     if ((h = getProbe()) == 0) {
>         // 初始化 probe
>         ThreadLocalRandom.current();
>         // h 对应新的 probe 值, 用来对应 cell
>         h = getProbe();
>         wasUncontended = true;
>     }
>     // collide 为 true 表示需要扩容
>     boolean collide = false;
>     for (;;) {
>         Cell[] as; Cell a; int n; long v;
>         // 已经有了 cells
>         if ((as = cells) != null && (n = as.length) > 0) {
>             // 还没有 cell
>             if ((a = as[(n - 1) & h]) == null) {
>                 // 为 cellsBusy 加锁, 创建 cell, cell 的初始累加值为 x
>                 // 成功则 break, 否则继续 continue 循环
>             }
>             // 有竞争, 改变线程对应的 cell 来重试 cas
>             else if (!wasUncontended)
>                 wasUncontended = true;
>             // cas 尝试累加, fn 配合 LongAccumulator 不为 null, 配合 LongAdder 为 null
>             else if (a.cas(v = a.value, ((fn == null) ? v + x : fn.applyAsLong(v, x))))
>                 break;
>             // 如果 cells 长度已经超过了最大长度, 或者已经扩容, 改变线程对应的 cell 来重试 cas
>             else if (n >= NCPU || cells != as)
>                 collide = false;
>             // 确保 collide 为 false 进入此分支, 就不会进入下面的 else if 进行扩容了
>             else if (!collide)
>                 collide = true;
>             // 加锁
>             else if (cellsBusy == 0 && casCellsBusy()) {
>                 // 加锁成功, 扩容
>                 continue;
>             }
>             // 改变线程对应的 cell
>             h = advanceProbe(h);
>         }
>         // 还没有 cells, 尝试给 cellsBusy 加锁
>         else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
>             // 加锁成功, 初始化 cells, 最开始长度为 2, 并填充一个 cell
>             // 成功则 break;
>         }
>         // 上两种情况失败, 尝试给 base 累加
>         else if (casBase(v = base, ((fn == null) ? v + x : fn.applyAsLong(v, x))))
>             break;
>     }
> }
> ```
>
> longAccumulate 流程图：
>
> ![longAccumulate流程图1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548979.PNG)
>
> ![longAccumulate流程图2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548950.PNG)
>
> 每个线程刚进入 longAccumulate 时，会尝试对应一个 cell 对象（找到一个坑位） 
>
> ![longAccumulate流程图3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548755.PNG)
>
> 获取最终结果通过 sum 方法
>
> ```java
> public long sum() {
>     Cell[] as = cells; Cell a;
>     long sum = base;
>     if (as != null) {
>         for (int i = 0; i < as.length; ++i) {
>             if ((a = as[i]) != null)
>                 sum += a.value;
>         }
>     }
>     return sum;
> }
> ```

### 5.8 Unsafe

**概述**

Unsafe 对象提供了非常底层的，操作内存、线程的方法，**Unsafe 对象不能直接调用，只能通过反射获得**

```java
public class UnsafeAccessor {

    static Unsafe unsafe;
    static {
        try {
            Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
            theUnsafe.setAccessible(true);
            unsafe = (Unsafe) theUnsafe.get(null);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            throw new Error(e);
        }
    }
    
    static Unsafe getUnsafe(){
        return unsafe;
    }
}
```

**Unsafe CAS操作**

```java
@Data
class Student{
    volatile int id;
    volatile String name;
}
```

```java
Unsafe unsafe = UnsafeAccessor.getUnsafe();
Field id = Student.class.getDeclaredField("id");
Field name = Student.class.getDeclaredField("name");
// 获得成员变量的偏移量
long idOffset = UnsafeAccessor.unsafe.objectFieldOffset(id);
long nameOffset = UnsafeAccessor.unsafe.objectFieldOffset(name);

Student student = new Student();
// 使用 cas 方法替换成员变量的值——>赋值时没有其他线程干扰，则成功赋值，否则得用while进行重试
UnsafeAccessor.unsafe.compareAndSwapInt(student, idOffset, 0, 20); // 返回ture
UnsafeAccessor.unsafe.compareAndSwapObject(student, nameOffset, null, "张三"); // 返回ture

System.out.println(student);
```

输出：

```java
Student(id=20, name=张三)
```

使用自定义的 AtomicData 实现之前线程安全的原子整数 Account 实现 

```java
class AtomicData{

    private volatile int data;
    static final Unsafe unsafe;
    static final Long DATA_OFFSET;

    static {
        unsafe = UnsafeAccessor.getUnsafe();

        try {
            // data 属性在 DataContainer 对象中的偏移量，用于 Unsafe 直接访问该属性
            DATA_OFFSET = unsafe.objectFieldOffset(AtomicData.class.getDeclaredField("data"));
        } catch (NoSuchFieldException e) {
            throw new Error(e);
        }
    }

    public AtomicData(int data){
        this.data = data;
    }

    public void decrease(int amount){
        int oldValue;
        while (true){
            // 获取共享变量旧值，可以在这一行加入断点，修改 data 调试来加深理解
            oldValue = data;
            // cas 尝试修改 data 为 旧值 + amount，如果期间旧值被别的线程改了，返回 false
            if(unsafe.compareAndSwapInt(this, DATA_OFFSET, oldValue, oldValue-amount)){
                return;
            }
        }
    }

    public int getData(){
        return data;
    }
}
```

Account实现

```java
Account.demo(new Account() {

    AtomicData atomicData = new AtomicData(10000);

    @Override
    public Integer getBalance() {
        return atomicData.getData();
    }

    @Override
    public void withdraw(Integer amount) {
        atomicData.decrease(amount);
    }
});
```

### 本章小结

- CAS 与 volatile
- API
  - 原子整数
  - 原子引用
  - 原子数组
  - 字段更新器
  - 原子累加器
- Unsafe
- 原理方面
  - LongAdder 源码
  - 伪共享 

