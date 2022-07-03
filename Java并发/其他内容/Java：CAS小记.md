# Java：CAS 小记

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

## 1. CAS 底层原理

### 概念

CAS 的全称是 Compare-And-Swap，它是 CPU 并发原语

它的功能是判断内存某个位置的值是否为预期值，如果是则更改为新的值，**这个过程是原子的**

CAS 并发原语体现在 Java 语言中就是 `sun.misc.Unsafe` 类的各个方法。调用 UnSafe 类中的 CAS 方法，JVM会帮我们实现出 CAS 汇编指令，这是一种完全依赖于硬件的功能，通过它实现了原子操作，再次强调，由于CAS 是一种系统原语，原语属于操作系统用于范畴，是由若干条指令组成，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，**也就是说 CAS 是一条 CPU 的原子指令，不会造成所谓的数据不一致的问题，也就是说 CAS 是线程安全的**。

### 代码使用

首先调用 AtomicInteger 创建了一个实例， 并初始化为 5

```java
// 创建一个原子类
AtomicInteger atomicInteger = new AtomicInteger(5);
```

然后调用CAS方法，企图更新成2019，这里有两个参数，一个是5，表示期望值，第二个就是我们要更新的值

```java
atomicInteger.compareAndSet(5, 2019)
```

然后再次使用了一个方法，同样将值改成1024

```java
atomicInteger.compareAndSet(5, 1024)
```

完整代码如下：

```java
/**
 * CASDemo
 *
 * 比较并交换：compareAndSet
 */
public class CASDemo {
    public static void main(String[] args) {
        // 创建一个原子类
        AtomicInteger atomicInteger = new AtomicInteger(5);

        /**
         * 一个是期望值，一个是更新值，但期望值和原来的值相同时，才能够更改
         * 假设三秒前，我拿的是5，也就是expect为5，然后我需要更新成 2019
         */
        System.out.println(atomicInteger.compareAndSet(5, 2019) + "\t current data: " + atomicInteger.get());

        System.out.println(atomicInteger.compareAndSet(5, 1024) + "\t current data: " + atomicInteger.get());
    }
}
```

上面代码的执行结果为：

```java
true	 current data: 2019
false	 current data: 2019
```

这是因为我们执行第一个的时候，期望值和原本值是满足的，因此修改成功，但是第二次后，主内存的值已经修改成了 2019，不满足期望值，因此返回了 false，本次写入失败

![cas过程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251602043.png)

这个就类似于 SVN 或者 Git 的版本号，如果没有人更改过，就能够正常提交，否则需要先将代码 pull 下来，合并代码后，然后提交。

### CAS 底层原理

首先我们先看看 `atomicInteger.getAndIncrement()` 方法的源码

```java
/**
 * Atomically increments by one the current value.
 *
 * @return the previous value
 */
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

从这里能够看到，底层又调用了一个 unsafe 类的 getAndAddInt 方法

#### 1. unsafe 类

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
}
```

Unsafe 是 CAS 的核心类，由于 Java 方法无法直接访问底层系统，需要通过本地（Native）方法来访问，**Unsafe 相当于一个后门，基于该类可以直接操作特定的内存数据。**Unsafe 类存在 sun.misc 包中，其内部方法操作可以像 C 的指针一样直接操作内存，因为 Java 中的 CAS 操作的执行依赖于 Unsafe 类的方法。

**注意Unsafe类的所有方法都是native修饰的，也就是说unsafe类中的方法都直接调用操作系统底层资源执行相应的任务**

为什么 Atomic 修饰的包装类，能够保证原子性，依靠的就是底层的 unsafe 类

#### 2. 变量 valueOffset

表示该变量值在内存中的偏移地址，因为 Unsafe 就是根据内存偏移地址获取数据的。

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
    // this:指当前对象
    // valueOffset:内存偏移量，也就是内存地址
}
```

从这里我们能够看到，通过 valueOffset，直接通过内存地址，获取到值，然后进行加1的操作

#### 3. 变量 value 用 volatile 修饰

这保证了多线程之间的内存可见性

```java
// AtomicInteger类源码
// setup to use Unsafe.compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe.getUnsafe();

public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

// Unsafe类源码：
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        // 不断循环拿到主内存中的值
        var5 = this.getIntVolatile(var1, var2);
        // var1:this, AtomicInteger 对象本身
        // var2:偏移量
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

* var5：就是我们从主内存中拷贝到工作内存中的值（每次都要从主内存拿到最新的值到自己的本地内存，然后执行 `compareAndSwapInt()`和主内存的值进行比较。因为线程不可以直接越过高速缓存，直接操作主内存，所以执行上述方法需要比较一次，再执行加1操作）

  那么操作的时候，需要比较工作内存中的值，和主内存中的值进行比较

  假设执行 `compareAndSwapInt()` 返回 false，那么就一直执行 while 方法，直到期望的值和真实值一样

* val1：AtomicInteger 对象本身
* var2：该对象值得引用地址
* var4：需要变动的数量
* var5：用 var1 和 var2 找到的内存中的真实值
  - 用该对象当前的值与 var5 比较
  - 如果相同，更新 var5 + var4 并返回 true
  - 如果不同，继续取值然后再比较，直到更新完成

这里没有用 synchronized，而用 CAS，这样**提高了并发性，也能够实现一致性**，是因为每个线程进来后，进入的 do while 循环，然后不断的获取内存中的值，判断是否为最新，然后在进行更新操作。

> 假设线程 A 和线程 B 同时执行 `getAndInt` 操作（分别跑在不同的CPU上）
>
> 1. AtomicInteger 里面的 value 原始值为3，即主内存中 AtomicInteger 的 value 为 3，根据 JMM 模型，线程A 和 线程B 各自持有一份值为3的副本，分别存储在各自的工作内存
> 2. 线程A 通过`getIntVolatile(var1 , var2)` 拿到 value 值3，这是 线程A 被挂起（该线程失去CPU执行权）
> 3. 线程B 也通过`getIntVolatile(var1, var2)`方法获取到 value 值也是3，此时刚好线程B没有被挂起，并执行了`compareAndSwapInt()`方法，比较内存的值也是3，成功修改内存值为4，线程B 打完收工，一切OK
> 4. 这时 线程A 恢复，执行 CAS 方法，比较发现自己手里的数字3和主内存中的数字4不一致，说明该值已经被其它线程抢先一步修改过了，那么 线程A 本次修改失败，只能够重新读取后在来一遍了，也就是在执行do while
> 5. 线程A 重新获取 value 值，因为变量 value 被 volatile 修饰，所以其它线程对它的修改，线程A 总能够看到，线程A 继续执行`compareAndSwapInt()`进行比较替换，直到成功。

**Unsafe类 + CAS思想： 也就是自旋，自我旋转**

### 底层汇编

Unsafe 类中的 `compareAndSwapInt()` 是一个本地方法，该方法的实现位于 `unsafe.cpp` 中

- 先想办法拿到变量 value 在内存中的地址
- 通过 `Atomic::cmpxchg` 实现比较替换，其中 `参数X` 是即将更新的值，`参数e` 是原内存的值

### CAS缺点

CAS不加锁，保证一致性，但是需要多次比较

- 循环时间长，开销大（因为执行的是 do while，如果比较不成功一直在循环，最差的情况，就是某个线程一直取到的值和预期值都不一样，这样就会无限循环）
- 只能保证一个共享变量的原子操作
  - 当对一个共享变量执行操作时，我们可以通过循环 CAS 的方式来保证原子操作
  - 但是对于多个共享变量操作时，循环 CAS 就无法保证操作的原子性，这个时候只能用锁来保证原子性
- 引出来 ABA 问题

### 总结

CAS 是 `compareAndSwap`，比较当前工作内存中的值和主物理内存中的值，如果相同则执行规定操作，否者继续比较直到主内存和工作内存的值一致为止

CAS有3个操作数，内存值V，旧的预期值A，要修改的更新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否者什么都不做。

##  2. 原子类AtomicInteger的ABA问题

### 连环套路

从 AtomicInteger 引出下面的问题：

CAS -> Unsafe -> CAS底层思想 -> ABA -> 原子引用更新 -> 如何规避ABA问题

### ABA问题是什么

狸猫换太子

![aba问题](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251602068.png)

假设现在有两个线程，分别是 T1 和 T2，然后T1执行某个操作的时间为10秒，T2执行某个时间的操作是2秒，最开始AB两个线程，分别从主内存中获取A值，但是因为B的执行速度更快，他先把A的值改成B，然后在修改成A，然后执行完毕，T1线程在10秒后，执行完毕，判断内存中的值为A，并且和自己预期的值一样，它就认为没有人更改了主内存中的值，就快乐的修改成B，但是实际上可能中间经历了 ABCDEFA 这个变换，也就是中间的值经历了狸猫换太子。

所以ABA问题就是，在进行获取主内存值的时候，该内存值在我们写入主内存的时候，已经被修改了N次，但是最终又改成原来的值了

### CAS导致ABA问题

CAS 算法实现了一个重要的前提，需要取出内存中某时刻的数据，并在当下时刻比较并替换，那么这个时间差会导致数据的变化。

比如说一个 线程1 从内存位置V中取出A，这时候另外一个 线程2 也从内存中取出A，并且 线程2 进行了一些操作将值变成了B，然后 线程2 又将V位置的数据变成A，这时候 线程1 进行CAS操作发现内存中仍然是A，然后 线程1 操作成功

**尽管线程1 的CAS操作成功，但是不代表这个过程就是没有问题的**

### ABA问题

CAS 只管开头和结尾，也就是头和尾是一样，那就修改成功，中间的这个过程，可能会被人修改过

### 原子引用

原子引用其实和原子包装类是差不多的概念，就是将一个 Java 类，用原子引用类进行包装起来，那么这个类就具备了原子性

```java
/**
 * 原子引用
 */
class User {
    String userName;
    int age;

    public User(String userName, int age) {
        this.userName = userName;
        this.age = age;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "userName='" + userName + '\'' +
                ", age=" + age +
                '}';
    }
}

public class AtomicReferenceDemo {

    public static void main(String[] args) {

        User z3 = new User("z3", 22);
        User l4 = new User("l4", 25);
        // 创建原子引用包装类
        AtomicReference<User> atomicReference = new AtomicReference<>();
        // 现在主物理内存的共享变量，为z3
        atomicReference.set(z3);
        
        // 比较并交换，如果现在主物理内存的值为z3，那么交换成l4
        System.out.println(atomicReference.compareAndSet(z3, l4) + "\t " + atomicReference.get().toString());

        // 比较并交换，现在主物理内存的值是l4了，但是预期为z3，因此交换失败
        System.out.println(atomicReference.compareAndSet(z3, l4) + "\t " + atomicReference.get().toString());
    }
}
```

### 基于原子引用的ABA问题

我们首先创建了两个线程，然后 T1线程，执行一次 ABA 的操作，T2线程 在 1s 后修改主内存的值

```java
/**
 * ABA问题代码体现
 */
public class ABADemo {

    /**
     * 普通的原子引用包装类
     */
    static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);

    public static void main(String[] args) {

        new Thread(() -> {
            // 把 100 改成 101 然后在改成100，也就是ABA
            atomicReference.compareAndSet(100, 101);
            atomicReference.compareAndSet(101, 100);
        }, "t1").start();

        new Thread(() -> {
            try {
                // 睡眠一秒，保证t1线程，完成了ABA操作
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 把100 改成 101 然后在改成100，也就是ABA
            System.out.println(atomicReference.compareAndSet(100, 2019) + "\t" + atomicReference.get());

        }, "t2").start();
    }
}
```

我们发现，它能够成功的修改，这就是 ABA 问题

### 解决ABA问题

新增一种机制，也就是修改版本号，类似于时间戳的概念

T1： 100 1 ---> 2019 2

T2： 100 1 ---> 101 2  ---> 100 3

如果T1修改的时候，版本号为2，落后于现在的版本号3，所以要重新获取最新值，这里就提出了一个使用时间戳版本号，来解决ABA问题的思路

### AtomicStampedReference

时间戳原子引用，来这里应用于版本号的更新，也就是每次更新的时候，需要比较期望值和当前值，**以及期望版本号和当前版本号**

```java
public class ABADemo {
    
    // 传递两个值，一个是初始值，一个是初始版本号
    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1);

    public static void main(String[] args) {

        System.out.println("============以下是ABA问题的解决==========");
        new Thread(() -> {
            // 获取版本号
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t 第一次版本号" + stamp);
            // 暂停t3一秒钟
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 传入4个值，期望值，更新值，期望版本号，更新版本号
            atomicStampedReference.compareAndSet(100, 101, atomicStampedReference.getStamp(), atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName() + "\t 第二次版本号" + atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet(101, 100, atomicStampedReference.getStamp(), atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName() + "\t 第三次版本号" + atomicStampedReference.getStamp());
        }, "t3").start();

        new Thread(() -> {
            // 获取版本号
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t 第一次版本号" + stamp);
            // 暂停t4 3秒钟，保证t3线程也进行一次ABA问题
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean result = atomicStampedReference.compareAndSet(100, 2019, stamp, stamp+1);
            System.out.println(Thread.currentThread().getName() + "\t 修改成功否：" + result + "\t 当前最新实际版本号：" + atomicStampedReference.getStamp());
            System.out.println(Thread.currentThread().getName() + "\t 当前实际最新值" + atomicStampedReference.getReference());
        }, "t4").start();
    }
}
```

运行结果为：

```java
============以下是ABA问题的解决==========
t3	 第一次版本号1
t4	 第一次版本号1
t3	 第二次版本号2
t3	 第三次版本号3
t4	 修改成功否：false	 当前最新实际版本号：3
t4	 当前实际最新值100
```

我们能够发现，线程t3，在进行 ABA 操作后，版本号变更成了3，而线程t4在进行操作的时候，就出现操作失败了，因为版本号和当初拿到的不一样

