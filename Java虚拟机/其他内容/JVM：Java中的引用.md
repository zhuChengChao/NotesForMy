# JVM：Java中的引用

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

在原来的时候，我们谈到一个类的实例化

```java
Person p = new Person()
```

在等号的左边，就是一个对象的引用，**存储在栈中**

而等号右边，就是实例化的对象，**存储在堆中**

其实这样的一个引用关系，就被称为**强引用**

## 整体架构

![java引用整体架构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251634822.png)

## 强引用

当内存不足的时候，JVM 开始垃圾回收，对于强引用的对象，就算是出现了 OOM 也不会对该对象进行回收，打死也不回收！

强引用是我们最常见的普通对象引用，只要还有一个强引用指向一个对象，就能表明对象还“活着”，垃圾收集器不会碰这种对象。在Java中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于**可达状态**，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会被用到，JVM 也不会回收，因此强引用是造成Java内存泄漏的主要原因之一。

对于一个普通的对象，如果没有其它的引用关系，只要超过了引用的作用域或者显示地将相应（强）引用赋值为null，一般可以认为就是可以被垃圾收集的了

> 当然具体回收时机还是要看垃圾回收策略

**强引用小例子：**

```java
public class StrongReferenceDemo {

    public static void main(String[] args) {
        // 这样定义的默认就是强应用
        Object obj1 = new Object();
        // 使用第二个引用，指向刚刚创建的Object对象
        Object obj2 = obj1;
        // 置空
        obj1 = null;
        // 垃圾回收
        System.gc();
        System.out.println(obj1);	// null
        System.out.println(obj2);	// java.lang.Object@1b6d3586
    }
}
```

输出结果我们能够发现，即使 `obj1` 被设置成了 null，然后调用 gc 进行回收，但是也没有回收实例出来的对象，`obj2` 还是能够指向该地址，也就是说垃圾回收器，并没有将该对象进行垃圾回收。

> **注意**：`System.gc();`调用该函数，**只是建议** JVM 可以进行垃圾回收了，但是不是一定会发生垃圾回收

## 软引用

软引用是一种相对弱化了一些的引用，需要用`Java.lang.ref.SoftReference`类来实现，可以让对象豁免一些垃圾收集，对于只有软引用的对象来讲：

- 当系统内存充足时，它不会被回收
- 当系统内存不足时，它会被回收

软引用通常在对内存敏感的程序中，比如高速缓存就用到了软引用，内存够用的时候就保留，不够用就回收

**例子：**

```java
public class SoftReferenceDemo {
    /**
     * 内存够用的时候
     */
    public static void softRefMemoryEnough() {
        // 创建一个强应用
        Object o1 = new Object();
        // 创建一个软引用
        SoftReference<Object> softReference = new SoftReference<>(o1);
        System.out.println(o1);		// 输出：java.lang.Object@1b6d3586
        System.out.println(softReference.get());  // 输出：java.lang.Object@1b6d3586

        o1 = null;
        // 手动GC
        System.gc();

        System.out.println(o1);		// 输出：null
        System.out.println(softReference.get());  // 输出：java.lang.Object@1b6d3586
    }

    /**
     * JVM配置，故意产生大对象并配置小的内存，让它的内存不够用了导致OOM，看软引用的回收情况
     * -Xms5m -Xmx5m -XX:+PrintGCDetails
     */
    public static void softRefMemoryNoEnough() {
        
        // 创建一个强应用
        Object o1 = new Object();
        // 创建一个软引用
        SoftReference<Object> softReference = new SoftReference<>(o1);
        System.out.println(o1);		// 输出：java.lang.Object@1b6d3586
        System.out.println(softReference.get());  // 输出：java.lang.Object@1b6d3586

        o1 = null;

        // 模拟OOM自动GC
        try {
            // 创建30M的大对象
            byte[] bytes = new byte[30 * 1024 * 1024];
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println(o1);	// 输出：null
            System.out.println(softReference.get());  // 输出：null
        }
    }

    public static void main(String[] args) {
        softRefMemoryEnough();
        softRefMemoryNoEnough();
    }
}
```

我们写了两个方法：一个是内存够用的时候，一个是内存不够用的时候

我们首先查看内存够用的时候，首先输出的是 `o1` 和软引用的`softReference`，我们都能够看到值

然后我们把`o1`设置为`null`，执行手动`GC`后，我们发现`softReference`的值还存在，说明内存充足的时候，软引用的对象不会被回收：

```bash
java.lang.Object@1b6d3586
java.lang.Object@1b6d3586
null
java.lang.Object@1b6d3586

[GC (System.gc()) [PSYoungGen: 3932K->712K(76288K)] 3932K->720K(251392K), 0.0005912 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 712K->0K(76288K)] [ParOldGen: 8K->599K(175104K)] 720K->599K(251392K), [Metaspace: 3222K->3222K(1056768K)], 0.0033044 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 76288K, used 3277K [0x000000076b000000, 0x0000000770500000, 0x00000007c0000000)
  eden space 65536K, 5% used [0x000000076b000000,0x000000076b333538,0x000000076f000000)
  from space 10752K, 0% used [0x000000076f000000,0x000000076f000000,0x000000076fa80000)
  to   space 10752K, 0% used [0x000000076fa80000,0x000000076fa80000,0x0000000770500000)
 ParOldGen       total 175104K, used 599K [0x00000006c1000000, 0x00000006cbb00000, 0x000000076b000000)
  object space 175104K, 0% used [0x00000006c1000000,0x00000006c1095ed0,0x00000006cbb00000)
 Metaspace       used 3229K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 349K, capacity 388K, committed 512K, reserved 1048576K
```

下面我们看当内存不够的时候，我们使用了JVM启动参数配置，给初始化堆内存为5M

```bash
-Xms5m -Xmx5m -XX:+PrintGCDetails
```

但是在创建对象的时候，我们创建了一个30M的大对象

```java
// 创建30M的大对象
byte[] bytes = new byte[30 * 1024 * 1024];
```

这就必然会触发垃圾回收机制，这也是中间出现的垃圾回收过程，最后看结果我们发现，`o1` 和 `softReference`都被回收了，因此说明：**软引用在内存不足的时候，会自动回收**

```bash
java.lang.Object@1b6d3586
java.lang.Object@1b6d3586
null	
null	# 较之前而言，此时softReference被回收了

[GC (Allocation Failure) [PSYoungGen: 1023K->488K(1536K)] 1023K->600K(5632K), 0.0007710 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1095K->488K(1536K)] 1207K->664K(5632K), 0.0005482 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 488K->488K(1536K)] 664K->700K(5632K), 0.0004621 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 488K->0K(1536K)] [ParOldGen: 212K->621K(4096K)] 700K->621K(5632K), [Metaspace: 3227K->3227K(1056768K)], 0.0033212 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] 621K->621K(5632K), 0.0002788 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] [ParOldGen: 621K->603K(4096K)] 621K->603K(5632K), [Metaspace: 3227K->3227K(1056768K)], 0.0035539 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

Heap
 PSYoungGen      total 1536K, used 95K [0x00000000ffe00000, 0x0000000100000000, 0x0000000100000000)
  eden space 1024K, 9% used [0x00000000ffe00000,0x00000000ffe17f78,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 4096K, used 603K [0x00000000ffa00000, 0x00000000ffe00000, 0x00000000ffe00000)
  object space 4096K, 14% used [0x00000000ffa00000,0x00000000ffa96c90,0x00000000ffe00000)
 Metaspace       used 3259K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 353K, capacity 388K, committed 512K, reserved 1048576K
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space  # 出现异常
	at SoftReferenceDemo.softRefMemoryNoEnough(SoftReferenceDemo.java:42)
	at SoftReferenceDemo.main(SoftReferenceDemo.java:56)
```

## 弱引用

弱引用：**不管内存是否够，只要有GC操作就会进行回收**

弱引用需要用 `java.lang.ref.WeakReference` 类来实现，它比软引用生存期更短

对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管 JVM 的内存空间是否足够，都会回收该对象占用的空间

**例子：**

```java
public class WeakReferenceDemo {
    public static void main(String[] args) {
        Object o1 = new Object();
        WeakReference<Object> weakReference = new WeakReference<>(o1);
        System.out.println(o1);
        System.out.println(weakReference.get());
        o1 = null;
        System.gc();
        System.out.println(o1);
        System.out.println(weakReference.get());
    }
}
```

我们看结果，能够发现，我们并没有制造出OOM内存溢出，而只是调用了一下GC操作，垃圾回收就把它给收集了

```
java.lang.Object@14ae5a5
java.lang.Object@14ae5a5

[GC (System.gc()) [PSYoungGen: 5246K->808K(76288K)] 5246K->816K(251392K), 0.0008236 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 808K->0K(76288K)] [ParOldGen: 8K->675K(175104K)] 816K->675K(251392K), [Metaspace: 3494K->3494K(1056768K)], 0.0035953 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

null
null
```

## 软引用 & 弱引用

**软引用和弱引用使用场景：**

场景：假如有一个应用需要读取大量的本地图片

- 如果每次读取图片都从硬盘读取则会严重影响性能
- 如果一次性全部加载到内存中，又可能造成内存溢出

此时使用软引用可以解决这个问题

设计思路：使用 HashMap 来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占的空间，从而有效地避免了 OOM 的问题

```java
Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();
```

**WeakHashMap 是什么？**

比如一些常常和底层打交道的 mybatis 等，底层都应用到了 WeakHashMap

WeakHashMap 和 HashMap 类似，只不过它的 Key 是使用了弱引用的，也就是说，当执行 GC 的时候，HashMap 中的 key 会进行回收，下面我们使用例子来测试一下

我们使用了两个方法，一个是普通的 HashMap 方法

我们输入一个 Key-Value 键值对，然后让它的 key 置空，然后在查看结果

```java
private static void myHashMap() {
    Map<Integer, String> map = new HashMap<>();
    Integer key = new Integer(1);
    String value = "HashMap";

    map.put(key, value);
    System.out.println(map);	// {1=HashMap}

    key = null;
    System.gc();
    
    System.out.println(map);	// {1=HashMap}
}
```

第二个是使用了`WeakHashMap`，完整代码如下

```java
private static void myWeakHashMap() {
    Map<Integer, String> map = new WeakHashMap<>();
    Integer key = new Integer(1);
    String value = "WeakHashMap";

    map.put(key, value);
    System.out.println(map);	// {1=WeakHashMap}

    key = null;
    System.gc();

    System.out.println(map);	// {}
}
```

从这里我们看到，对于普通的HashMap来说，key置空并不会影响，HashMap的键值对，因为这个属于强引用，不会被垃圾回收；

但是WeakHashMap，在进行GC操作后，弱引用的就会被回收

## 虚引用

虚引用又称为幽灵引用，需要`java.lang.ref.PhantomReference` 类来实现

顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。

**如果一个对象持有虚引用，那么它就和没有任何引用一样**，在任何时候都可能被垃圾回收器回收，它不能单独使用也不能通过它访问对象，**虚引用必须和引用队列ReferenceQueue联合使用**。

**虚引用的主要作用是跟踪对象被垃圾回收的状态**，仅仅是提供一种确保对象被 finalize 以后，做某些事情的机制。

`PhantomReference`的`get`方法总是返回`null`，因此无法访问对象的引用对象。其意义在于说明一个对象已经进入 finalization 阶段，可以被 gc 回收，用来实现比 finalization 机制更灵活的回收操作

换句话说，设置虚引用关联**唯一目的**，就是在**这个对象被收集器回收的时候，收到一个系统通知或者后续添加进一步的处理**，Java技术允许使用finalize()方法在垃圾收集器将对象从内存中清除出去之前，做必要的清理工作

这个就相当于Spring AOP里面的**后置通知**

**场景：一般用于在回收时候做通知相关操作**

## ReferenceQueue：引用队列

软引用，弱引用，虚引用在回收之前，都可以用引用队列保存一下

我们在初始化的弱引用或者虚引用的时候，可以传入一个引用队列

```java
Object o1 = new Object();

// 创建引用队列
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

// 创建一个弱引用，传入引用队列
WeakReference<Object> weakReference = new WeakReference<>(o1, referenceQueue);
```

那么在进行 GC 回收的时候，弱引用和虚引用的对象都会被回收，但是在回收之前，它会被送至引用队列中

完整代码如下：

```java
public class PhantomReferenceDemo {

    public static void main(String[] args) {
        Object o1 = new Object();

        // 创建引用队列
        ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

        // 创建一个弱引用
        WeakReference<Object> weakReference = new WeakReference<>(o1, referenceQueue);
        // 创建一个虚引用
        PhantomReference<Object> phantomReference = new PhantomReference<>(o1, referenceQueue);

        System.out.println(o1);						 // java.lang.Object@1b6d3586
        System.out.println(weakReference.get());	 // java.lang.Object@1b6d3586
        System.out.println(phantomReference.get());  // null:虚引用的get方法总是返回null
        // 取队列中的内容
        System.out.println(referenceQueue.poll());   // null
        System.out.println(referenceQueue.poll());	 // null, 没有进行GC，队列中为空

        o1 = null;
        System.gc();
        System.out.println("执行GC操作");

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(o1);						// null
        System.out.println(weakReference.get());	// null
        System.out.println(phantomReference.get());	// null
        // 取队列中的内容
        System.out.println(referenceQueue.poll());	// 发生了GC，引用队列中有内容了
        System.out.println(referenceQueue.poll());
    }
}
```

运行结果

```bash
java.lang.Object@1b6d3586
java.lang.Object@1b6d3586
null
null
null
执行GC操作
null
null
null
java.lang.ref.WeakReference@4554617c
java.lang.ref.PhantomReference@74a14482
```

从这里我们能看到，在进行垃圾回收后，我们弱引用对象，也被设置成null，但是在队列中还能够导出该引用的实例，这就说明在回收之前，该弱引用的实例被放置引用队列中了，我们可以通过引用队列进行一些后置操作

## GCRoots和四大引用小总结

- 红色部分在垃圾回收之外，也就是强引用的
- 蓝色部分：属于软引用，在内存不够的时候，才回收
- 虚引用和弱引用：每次垃圾回收的时候，都会被干掉，但是它在干掉之前还会存在引用队列中，我们可以通过引用队列进行一些通知机制

![四大引用小结](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251634452.png)