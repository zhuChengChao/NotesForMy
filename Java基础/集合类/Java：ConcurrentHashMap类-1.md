# image/(概述)

> 对 Java 中的 **ConcurrentHashMap类**，做一个微不足道的小小小小记，分三篇博客，本文为第一篇。

## 概述

ConcurrentHashMap 是 Java 并发包中提供的一个线程安全且高效的 HashMap 实现，其在并发编程的场景中使用频率非常之高

众所周知，哈希表是种非常高效，复杂度为 O(1) 的数据结构，在Java开发中，我们最常见到最频繁使用的就是 HashMap 和 HashTable，但是在线程竞争激烈的并发场景中使用都不够合理。

> 关于 HashMap 和 HashTable 的进一步了解，可见：[Java：HashMap类小记](https://www.cnblogs.com/zhuchengchao/p/14298141.html)，[Java：HashTable类小记](https://www.cnblogs.com/zhuchengchao/p/14444323.html) 这两篇博文

这里就先谈一下 HashMap 和 HashTable 与 ConcurrentHashMap 的区别所在：

### HashMap 与 ConcurrentHashMap 区别：

> 后续指的是 JDK1.7 中的 ConcurrentHashMap

HashMap 不是线程安全的，而 ConcurrentHashMap **是线程安全的**；

ConcurrentHashMap 采用**锁分段技术**，将整个 Hash 桶进行了分段 segment，也就是将这个大的数组分成了几个小的片段 segment，而且每个小的片段 segment 上面都有锁存在，那么在插入元素的时候就需要先找到应该插入到哪一个片段 segment，然后再在这个片段上面进行插入，虽然这里还需要获取 segment 锁，但这样做明显减小了锁的粒度。

### HashTable 和 ConcurrentHashMap 的区别：

HashTable 和 ConcurrentHashMap 相比，**效率低**。 Hashtable 之所以效率低主要是使用了 synchronized 关键字对 put 等操作进行加锁，而 synchronized 关键字加锁是对整张 Hash 表的，即每次锁住整张表让线程**独占**，致使效率低下，而 ConcurrentHashMap 在对象中保存了一个 Segment 数组，即将整个 Hash 表划分为多个分段；而每个Segment 元素，即每个分段则类似于一个 Hashtable；这样，在执行 put 操作时首先根据 hash 算法定位到元素属于哪个 Segment，然后对该 Segment 加锁即可，因此， ConcurrentHashMap 在多线程并发编程中可以实现多线程 put 操作。

![ConcurrentHashMap分段锁](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251613740.JPG)

**总结：**

ConcurrentHashMap 的出现也意味着 HashTable 的落幕，所以在以后的项目中，尽量少用 HashTable；

对于普通场景可以使用 HashMap 实现，如果是高并发场景建议使用 ConcurrentHashMap 实现

## 实现原理

> 这里仅进行一个简单的说明，后续对于 ConcurrentHashMap 在 JDK7 与 JDK8 中的实现会做进一步分析

### JDK 7 中的 ConcurrentHashMap

> JDK7 中 ConcurrentHashMap 采用了 **数组 + Segment + 分段锁** 的方式实现。
>
> 以下内容主要参考：https://www.cnblogs.com/chengxiao/p/6842045.html，稍微了解了一下

ConcurrentHashMap 采用了非常精妙的"分段锁"策略，ConcurrentHashMap 的主干是个 Segment 数组。

```java
final Segment<K,V>[]  segments;
```

Segment 继承了 ReentrantLock，所以它就是一种可重入锁（ReentrantLock)。在 ConcurrentHashMap，一个 Segment 就是一个子哈希表，Segment 里维护了一个 HashEntry 数组，并发环境下，对于不同 Segment 的数据进行操作是不用考虑锁竞争的。就按默认的 ConcurrentLevel 为 16 来讲，理论上就允许 16 个线程并发执行。

所以，**对于同一个 Segment 的操作才需考虑线程同步，不同的 Segment 则无需考虑**。Segment 类似于 HashMap，一个 Segment 维护着一个HashEntry 数组：

```java
transient volatile  HashEntry<K,V>[]  table;
```

HashEntry 是目前我们提到的最小的逻辑处理单元了。**一个 ConcurrentHashMap 维护一个 Segment 数组，一个 Segment 维护一个 HashEntry 数组。**

因此，ConcurrentHashMap 定位一个元素的过程需要进行两次 Hash 操作。第一次 Hash 定位到 Segment，第二次 Hash 定位到元素所在的链表的头部。

### JDK 8 中的 ConcurrentHashMap

> JDK 8：中 ConcurrentHashMap 参考了 JDK 8 HashMap 的实现，采用了 **数组 + 链表 + 红黑树** 的实现方式来设计，**内部大量采用 CAS 操作**。
>

**这里仅做一个总结，进一步分析见后续博文**

1. JDK1.8 取消了 segment 数组，**直接用 table 保存数据，锁的粒度更小**，减少并发冲突的概率。

   ```java
   // 所有数据都存在table中, 只有当第一次插入时才会被加载，扩容时总是以2的倍数进行
   transient volatile Node<K,V>[] table;
   ```

2. JDK1.8 存储数据时采用了**链表+红黑树**的形式，纯链表的形式时间复杂度为O(n)，红黑树则为 O(logn)，性能提升很大。

   > 什么时候链表转红黑树？
   >
   > * 当在一个桶中元素形成的链表中元素个数超过8个的时候；
   > * table的size大于64

3. JDK1.8 的实现降低锁的粒度，JDK1.7版本锁的粒度是**基于Segment的，包含多个 HashEntry**，而JDK1.8锁的粒度**就是HashEntry（首节点）**

4. JDK1.8 版本的数据结构变得更加简单，使得操作也更加清晰流畅，**因为已经使用 synchronized 来进行同步**，所以不需要分段锁的概念，也就不需要 Segment 这种数据结构了，由于粒度的降低，实现的复杂度也增加了

5. JDK1.8 使用**红黑树来优化链表**，基于长度很长的链表的遍历是一个很漫长的过程，而红黑树的遍历效率是很快的，代替一定阈值的链表，这样形成一个最佳拍档

6. **JDK1.8 为什么使用内置锁synchronized来代替重入锁ReentrantLock，我觉得有以下几点**

   1. 因为粒度降低了，在相对而言的低粒度加锁方式，synchronized并不比ReentrantLock差，在粗粒度加锁中ReentrantLock可能通过Condition来控制各个低粒度的边界，更加的灵活，而在低粒度中，Condition的优势就没有了
   2. JVM的开发团队从来都没有放弃synchronized，而且基于JVM的synchronized优化空间更大，使用内嵌的关键字比使用API更加自然
   3. 在大量的数据操作下，对于JVM的内存压力，基于API的ReentrantLock会开销更多的内存，虽然不是瓶颈，但是也是一个选择依据

## Unsafe类

由于在ConcurrentHashMap中，为了避免进行加锁操作，使用了大量的CAS操作，而CAS操作在Java中是通过 Unsafe 类进行实现的，故在此先分析一波 Unsafe 类

### 概述

Unsafe 类相当于是一个 Java 语言中的后门类，**提供了硬件级别的原子操作**，所以在一些并发编程中被大量使用。JDK 已经作出说明，该类对程序员而言不是一个安全操作，在后续的JDK升级过程中，可能会禁用该类。所以这个类的使用是一把双刃剑，实际项目中谨慎使用，以免造成JDK升级不兼容问题。

### API

> 这个类是获取对象的偏移情况，而在ConcurrentHashMap中体现的就是数组对象，后续API中就以数组来进行说明

- `public native int arrayBaseOffset(Class<?> var1)`：获取数组的基础偏移量；
- `public native int arrayIndexScale(Class<?> var1)`：获取数组中元素的偏移间隔，要获取对应所以的元素，将索引号和该值相乘，获得数组中指定角标元素的偏移量
- `public native Object getObjectVolatile(Object var1, long var2)`：获取对象上的属性值或者数组中的元素
- `public native Object getObject(Object var1, long var2)`：获取对象上的属性值或者数组中的元素，已过时
- `public native void putOrderedObject(Object var1, long var2, Object var4)`：设置对象的属性值或者数组中某个角标的元素，**更高效，但不保证线程间即时可见性**
- `public native void putObjectVolatile(Object var1, long var2, Object var4)`：设置对象的属性值或者数组中某个角标的元素，**volatile 保证线程间及时可见性**
- `public native void putObject(Object var1, long var2, Object var4);`：设置对象的属性值或者数组中某个角标的元素，已过时

### 代码演示

```java
public class Demo {

    public static void main(String[] args) throws Exception {

        // 数组对象
        Integer[] arr = {2,5,1,8,10};
        // 暴力反射获取Unsafe对象
        Unsafe unsafe = getUnsafe();
        // 获取Integer[]的基础偏移量
        int baseOffset = unsafe.arrayBaseOffset(Integer[].class);
        // 获取Integer[]中元素的偏移间隔
        int indexScale = unsafe.arrayIndexScale(Integer[].class);

        // 获取数组中索引为2的元素对象:baseoffset + 2*indexScale
        Object o = unsafe.getObjectVolatile(arr, (2 * indexScale) + baseOffset);
        System.out.println(o); // 1

        // 设置数组中索引为2的元素值为100
        unsafe.putOrderedObject(arr,(2 * indexScale) + baseOffset,100);

        System.out.println(Arrays.toString(arr)); // [2, 5, 100, 8, 10]
    }

    // 反射获取Unsafe对象
    public static Unsafe getUnsafe() throws Exception {
        // 获取Unsafe类无法直接通过Unsafe的内置函数getUnsafe来获取
        Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
        // 暴力反射
        theUnsafe.setAccessible(true);
        return (Unsafe) theUnsafe.get(null);
    }
}
```

### 图解说明

![数组偏移量计算](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251613734.PNG)

## 参考

https://space.bilibili.com/488677881

☆ConcurrentHashMap实现原理及源码分析☆：https://www.cnblogs.com/chengxiao/p/6842045.html

☆关于jdk1.8中ConcurrentHashMap的方方面面☆：https://blog.csdn.net/tp7309/article/details/76532366

https://mp.weixin.qq.com/s/q1r9Pno6ANUzZ9wMzA-JSg

http://www.apgblogs.com/hashmap-hashtable-concurrenthashmap/
