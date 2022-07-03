# Java：ConcurrentHashMap类-2(JDK7)

> 对 Java 中的 **ConcurrentHashMap类**，做一个微不足道的小小小小记，分三篇博客，本文为第二篇。

## 结构说明

### 构造函数

**无参构造：默认调用带参构造函数**

```java
// 空参构造
public ConcurrentHashMap() {
    // 调用本类的带参构造，都是使用的默认值
    //   DEFAULT_INITIAL_CAPACITY = 16
    //   DEFAULT_LOAD_FACTOR = 0.75f
    //   int DEFAULT_CONCURRENCY_LEVEL = 16
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
}
```

**带参数构造**

**下方代码总结**：ConcurrentHashMap中保存了一个**默认长度为16的Segment[]**，每个Segment元素中保存了一个**默认长度为2的HashEntry[]数组**，我们添加的元素，是存入对应的Segment中的HashEntry[]中。所以ConcurrentHashMap中默认元素的长度是32个，而不是16个

```java
// initialCapacity定义ConcurrentHashMap存放元素的容量
// concurrencyLevel定义ConcurrentHashMap中Segment[]的大小
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    // 参数校验
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
   		concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    int sshift = 0;
    int ssize = 1;    // 指的是Segment数组的长度
    // 计算Segment[]的大小，保证是2的幂次方数
    while (ssize < concurrencyLevel) {
        ++sshift;
        // 左移1位，即*2操作，最终保证：ssize为大于等于concurrencyLevel的2的幂次方数
        ssize <<= 1;  
    }
    // 这两个值用于后面计算Segment[]的角标
    this.segmentShift = 32 - sshift;  // a.假定concurrencyLevel为16，则sshift为4,则segmentShift为28
    this.segmentMask = ssize - 1;  // b.假定concurrencyLevel为16,则ssize为16，segmentMask为15
    
    if(initialCapacity > MAXIMUM_CAPACITY)
    	initialCapacity = MAXIMUM_CAPACITY
    // 计算每个Segment中存储元素的个数，基于上述假定，16/16=1，即c=1
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    // 最小Segment中存储元素的个数为2,MIN_SEGMENT_TABLE_CAPACITY=2
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    // 矫正每个Segment中存储元素的个数，保证是2的幂次方，最小为2
    while (cap < c)
        cap <<= 1;
    // 创建一个Segment对象，作为其他Segment对象的模板
    Segment<K,V> s0 =
        // new了一个HashEntry数组，容量即为cap
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);  
    // 创建一个Segment数组
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    // 利用Unsafe类，将创建的Segment对象存入0角标位置
    // 通过cas，将创建的Segment对象设置到Segment[]数组中
    // 其中SBASE即为数组的基础偏移量，baseOfset
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    // final Segment<K,V>[] segments;
    this.segments = ss;
}
```

### 内部类

**Segment 数组**

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
	
    // 构造函数
    Segment(float 1f, int threshold, HashEntry<K,V>[] tab){
        this.loadFactor = 1f;
        this.threshold = threshold;
        this.table = tab;
    }
    
    // ...
}
```

**Segment 是继承自 ReentrantLock 的**，它可以实现同步操作，从而保证多线程下的安全。因为每个Segment 之间的锁互不影响，所以我们也将ConcurrentHashMap中的这种锁机制称之为**分段锁**，这比HashTable的线程安全操作高效的多。

**HashEntry 数组**

```java
// ConcurrentHashMap中真正存储数据的对象
static final class HashEntry<K,V> {
    final int hash;    // 通过运算，得到的键的hash值
    final K key;       // 存入的键
    volatile V value;  // 存入的值
    volatile HashEntry<K,V> next;  // 记录下一个元素，形成单向链表

    HashEntry(int hash, K key, V value, HashEntry<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

### 图示Segment数组

![segment数组](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251613443.PNG)

## put 过程

### 源码分析

`put()` 方法：

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        // ConcurrentHashMap的value不能为null
        throw new NullPointerException();
    // 基于key，计算hash值，key也不允许为null，hash中会调用key的hashcode方法
    int hash = hash(key);
    // 因为一个键要计算两个数组的索引，为了避免冲突，这里取hash的高位计算Segment[]的索引，后续进一步说明
    int j = (hash >>> segmentShift) & segmentMask;
    // a)根据角标位获取segments数组中的segment元素，通过cas方式获取，计算方式如下:
    // j<<SSHIFT+SBASE 等价于  j*scale+sbase，这样操作是考虑到了效率问题
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        // 判断该索引位的Segment对象是否创建，没有就创建，☆后续分析☆
        s = ensureSegment(j);
    // 调用Segment的put方法实现元素添加，☆后续分析☆
    return s.put(key, hash, value, false);
}

// 进一步对索引计算的说明：
(hash >>> segmentShift) & segmentMask;
// 还是按照初始的concurrencyLevel为16为16，则segmentShift为28，segmentMask为15
// 对hash值无符号右移动segmentShift，即取到了hash的高4位
// 将hash值的高4位和segmentMask进行按位 &, 等价于(Segment的数组长度-1)取模，计算出该hash值在Segment数组的角标位
```

初始对于角标位置的segment元素：`ensureSegment()`

```java
// 创建对应索引位的Segment对象，并返回
// 在创建segment元素并放入HashEntry数组的过程中并没有加锁，而是通过cas保证了线程安全
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    // 同样获取segments中的偏移量
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    // 获取，如果为null，即创建，
    // 而获取这个segment元素判空在进入函数之前已经进行了一次，在此处再一次获取则是考虑到了线程安全问题
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        // 以0角标位的Segment为模板
        Segment<K,V> proto = ss[0]; // use segment 0 as prototype
        int cap = proto.table.length;  
        float lf = proto.loadFactor;
        int threshold = (int)(cap * lf);
        // 创建了HashEntry[]
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        // 获取，如果为null，即创建，又去了一次角标位置的segment元素，也是考虑了线程安全
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
            == null) { // recheck
            // 正在创建对应的segment
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // 自旋方式，将创建的Segment对象放到Segment[]中，确保线程安全
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                // 当待放的位置为null时，则放入
                // 如果放成功了，则break
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    // 不为空了则返回
    return seg;
}
```

当 segments 数组中存在了 HashEntry 时，则通过调用 Segment 的 put 方法实现元素添加

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // tryLock尝试获取锁，获取成功，node为null，代码向下执行
    // 如果有其他线程占据锁对象，那么去做别的事情，而不是一直等待，这是为了提升效率，
    // 即调用方法scanAndLockForPut☆后续分析☆
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        // 成功进入了，说明获取Segments数组对应的Segment元素成功了，即成功加锁
        HashEntry<K,V>[] tab = table;
        // 取hash的低位，计算HashEntry[]的索引
        int index = (tab.length - 1) & hash;
        // 获取HashEntry[]索引位的元素对象，也是通过cas的方式获取
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {  // 死循环
            if (e != null) {  // 获取的元素对象不为空，发生了hash冲突
                K k;
                // 如果是重复元素，覆盖原值
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;  // 替换成功之后break
                }
                // 如果不是重复元素，获取链表的下一个元素，继续循环遍历链表
                e = e.next;
            }
            else { // 如果获取到的元素为空
                // 若当前添加的键值对的HashEntry对象已经创建
                if (node != null)
                    node.setNext(first);  // 头插法关联即可：即当前节点的下一个节点为当前的头节点
                else
                    // 创建当前添加的键值对的HashEntry对象
                    node = new HashEntry<K,V>(hash, key, value, first);
                // 添加的元素数量递增
                int c = count + 1;
                // 判断是否需要扩容
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    // 需要扩容，则进行扩容操作，☆后续分析☆
                    rehash(node);
                else
                    // 不需要扩容
                    // 将当前添加的元素对象，存入数组角标位，完成头插法添加元素
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        //释放锁
        unlock();
    }
    return oldValue;
}
```

在 put 方法中，若没有拿获取到锁的情况下，则调用方法`scanAndLockForPut()`，去完成HashEntry对象的创建，提升效率

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    // 获取头部元素，获取是可以获取的
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null；
    int retries = -1; // negative while locating node
    while (!tryLock()) {
        // 获取锁失败，只有当获取锁成功后才会跳出循环，一种自旋操作
        HashEntry<K,V> f; // to recheck first below
        // 刚刚开始肯定是进入这里
        if (retries < 0) {
            // 没有下一个节点，并且也不是重复元素，直接创建HashEntry对象，不再遍历
            if (e == null) {
                if (node == null) // speculatively create node
                    // 创建了HashEntry对象
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                // 是一个重复元素，不需要创建HashEntry对象，不再遍历
                retries = 0;
            else
                // 继续遍历下一个节点
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            // 如果尝试获取锁的次数过多，直接阻塞
            // MAX_SCAN_RETRIES会根据可用cpu核数来确定
            lock();  // 当自旋的次数大于MAX_SCAN_RETRIES，则不再自旋了，直接上锁（阻塞锁），直到获取到锁之后再break
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            // 如果期间有别的线程获取锁，导致HashEntry结构发生变化，则重新遍历
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    // 成功获取到锁了，则退出该函数，继续回到put方法中
    return node;
}
```

### 代码演示

> 这里“通话”和“重地”的哈希值是一样的，那么他们添加时，会存入同一个Segment对象，必然会存在锁竞争

```java
public static void main(String[] args) throws Exception {
    final ConcurrentHashMap chm = new ConcurrentHashMap();

    new Thread(){
        @Override
        public void run() {
            chm.put("通话","11");
            System.out.println("-----------");
        }
    }.start();

	//让第一个线程先启动，进入put方法
    Thread.sleep(1000);

    new Thread(){
        @Override
        public void run() {
            chm.put("重地","22");
            System.out.println("===========");
        }
    }.start();
}
```

> 注意：在Debug时，在断点处需要选择线程断点，且加上响应的条件，这里就不再演示了

## 扩容 rehash

### 源码分析

```java
// 这个rehash操作都是在获取锁的情况下进行的，因此是线程安全的
private void rehash(HashEntry<K,V> node) {
    // 获取原数组和原容量
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    // 两倍容量
    int newCapacity = oldCapacity << 1;
    // 计算新的阈值
    threshold = (int)(newCapacity * loadFactor);
    // 基于新容量，创建HashEntry数组
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    // 计算新的mask，这个mask当然是用于定位Segment的
    int sizeMask = newCapacity - 1;
   	// 实现数据迁移，遍历oldtable
    for (int i = 0; i < oldCapacity ; i++) {
        // 获取该位置的首节点
        HashEntry<K,V> e = oldTable[i];
        // 该位置有节点，则需要进行迁移操作
        if (e != null) {
            HashEntry<K,V> next = e.next;
            // idx表示的是新的位置
            int idx = e.hash & sizeMask;
            if (next == null)   //  Single node on list
                // 原位置只有一个元素，直接放到新数组即可
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                // 这里的迁移方式做了一定的优化操作和hashmap迁移相比的话
                // === 图1 ===
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // === 图1 ===
                
                // === 图2 ===
                newTable[lastIdx] = lastRun;
                // === 图2 ===
                // Clone remaining nodes
                // === 图3 ===
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    // 这里旧的HashEntry不会放到新数组
                    // 而是基于原来的数据创建了一个新的 HashEntry 对象，放入新数组
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
                // === 图3 ===
            }
        }
    }
    //采用头插法，将新元素加入到数组中
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

上述说道了，在数据迁移方式中做了一定的优化操作和hashmap迁移相比的话，用个人的话概括而言如下：**部分连接在一起的节点数据若重hash后节点idx不变，则直接迁移，而不需要再重新创建节点，即存在一种情况下同时迁移多个节点数据，减少了迁移的次数**，能提升迁移的效率，见后续图解部分说明。

### 图解

图1：

![chm的rehash1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251613767.PNG)

图2：

![chm的rehash2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251613589.PNG)

图3：

![chm的rehash3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251613161.PNG)

优化图解：

![chm的rehash优化说明](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251613610.PNG)

## 集合长度获取

也做了一定的操作保证了 size 是安全的，具体见源码分析

```java
public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    // 拿到Segments数据
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum;         // sum of modCounts
    long last = 0L;   // previous sum
    // 重试次数
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {  // 死循环
            // 当第4次走到这个地方时，会将整个Segment[]的所有Segment对象锁住，此时结果必然是正确的
            // 第4次是因为RETRIES_BEFORE_LOCK=2
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    // 累加所有Segment的操作次数，是为了判断在获取长度时，集合是否发生了变化
                    sum += seg.modCount;
                    // segment的长度
                    int c = seg.count;
                    // 累加所有segment中的元素个数 size+=c
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            // 当这次累加值和上一次累加值一样，证明没有进行新的增删改操作，返回sum
            // 第一次last为0，如果有元素的话，这个for循环最少循环两次的，保证了结果正确
            if (sum == last)
                break;
            // 记录累加的值
            last = sum;
        }
    } finally {
        // 如果之前有锁住，解锁
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    // 溢出，返回int的最大值，否则返回累加的size
    return overflow ? Integer.MAX_VALUE : size;
}
```

## 参考

这部分内容主要参考的是 bilibili UP 主：https://space.bilibili.com/488677881 讲解的 ConcurrentHashMap

其具体的视频地址未给出，而是通过其提供的网盘地址进行下载的
