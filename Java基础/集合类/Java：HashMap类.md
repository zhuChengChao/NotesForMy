# Java：HashMap类

> 对 Java 中的 **HashMap类**，做一个微不足道的小小小小记

## 概述

**HashMap**：存储数据采用的哈希表结构，元素的存取顺序不能保证一致。由于要保证键的唯一、不重复，需要重写键的`hashCode()`方法、`equals()`方法。

Map 接口中定义了很多方法，常用的如下：

* `public V put(K key, V value)` : 把指定的键与指定的值添加到Map集合中。
* `public V remove(Object key)` : 把指定的键所对应的键值对元素 在Map集合中删除，返回被删除元素的值。
* `public V get(Object key)` : 根据指定的键，在Map集合中获取对应的值。
* `public Set<K> keySet()` : 获取Map集合中所有的键，存储到Set集合中。
* `public Set<Map.Entry<K,V>> entrySet()` : 获取到Map集合中所有的键值对对象的集合(Set 集合)。

**部分源代码**：（JDK 1.8）

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    
    private static final long serialVersionUID = 362498820763181265L;
    
    // 默认的初始容量 16 
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    // 最大容量 2^30
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 负载因子，代表了table的填充度有多少，默认是0.75
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 转换为红黑树的阈值，大于则转换为红黑树
    static final int TREEIFY_THRESHOLD = 8;
    // 红黑树转为链表的阈值,会在resize时还原
    static final int UNTREEIFY_THRESHOLD = 6;
    // 当哈希表中的容量 > 该值时，才允许树形化链表
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    // 静态内部类 后续分析 key-value都用Node来进行了封装
    static class Node<K,V> implements Map.Entry<K,V> {
        // ...
    }
    // 静态内部类 红黑树节点类型，且继承了LinkedHashMap.Entry，还维护一个双向链表
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        // ...
    }
    
    /* ---------------- Static utilities -------------- */
    // ...
    
    /* ---------------- Fields -------------- */
    // 主干数组，Node数组
    transient Node<K,V>[] table;
    // 就是HashMap中各个键值对映射关系的集合，可以通过entrySet()方法输出
    // entrySet中定义的类型为Map.Entry，实际存储的是为HashMap.Node
    // 作用：方便遍历HashMap
    transient Set<Map.Entry<K,V>> entrySet;
    // 实际存储的key-value键值对的个数
    transient int size;
    // 结构化修改次数，和fail-fast相关
    transient int modCount;
   	// threshold一般为 capacity*loadFactory,与扩容相关
    int threshold;
    // 负载因子
    final float loadFactor;
}
```

## 数据结构

### JDK1.7

**JDK1.7：Entry数组 + 单向链表**

HashMap 的主干是一个 Entry 数组。Entry 是 HashMap 的基本组成单元，每一个 Entry 包含一个 key-value 键值对。

```java
// HashMap的主干数组，可以看到就是一个Entry数组，初始值为空数组{}
// 且主干数组的长度一定是2的次幂，至于为什么这么做，后续会进行分析
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```

Entry 是 HashMap 中的一个静态内部类。

代码如下：

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;  // 存储指向下一个Entry的引用，单链表结构
    int hash;         // 对key的hashcode值进行hash运算后得到的值，存储在Entry，避免重复计算

    /**
     * Creates new entry. 构造函数
     */
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    } 
}

// 插入节点：头插法
void createEntry(int hash, K key, V value, int bucketIndex) {
    // 取出索引位置的元素
    Entry<K,V> e = table[bucketIndex];
    // 将新的元素放置到索引位，同时将原来的作为新元素的下一个保存，形成单向链表
    // 头插法
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```

有下图：

![hashmap](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251614371.JPG)

**总结**：

简单来说：**HashMap 由数组+链表组成的**，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；

如果定位到的数组位置不含链表（当前entry的next指向null），那么对于查找，添加等操作很快，仅需一次寻址即可；

如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过 key 对象的 equals 方法逐一比对查找。

所以，性能考虑，HashMap中的链表出现越少，性能才会越好。

### JDK1.8

> 大体上还是和 JDK 1.7 中所体现的相似

**JDK1.8：Node数组 + 链表/红黑树；**

> 进一步说明：
>
> 1. Node数组：这里变成了 Node 数组，而不是 Entry 数组了，Node 数组其实实现了 Entry 数组，见源码部分
> 2. 链表：单向链表
> 3. **红黑树：红黑树节点+双向链表**

Entry 和 Node 都包含 key、value、hash、next 属性，以下是 **Node 的源码**：

```java
// HashMap的主干数组
transient Node<K,V>[] table;

// Node类继承了 Map.Entry<K,V>，是Map接口中的一个接口
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;  // 存储了key的hash值
    final K key;
    V value;
    Node<K,V> next;  // 指向下一个节点的指针

    Node(int hash, K key, V value, Node<K,V> next) {
        // 构造函数
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
	
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        // 注意这是获取Node的hash值
        // 操作：对key和value取了一个异或操作
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        // 判断是否相等，同时对key和value进行判断
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

以下是**红黑树的节点类源码**：

```java
// 红黑树节点+双向链表，可见其继承了LinkedHashMap.Entry<K,V>
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;	   // 左子树
    TreeNode<K,V> right;   // 右子数
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
    
    // ...关于树的一些操作
}

// 红黑树中的双向链表就体现在此
// 而之所以在维护红黑树的同时，再维护了一个双向链表的结构，这是为了扩容方便
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;  // 前一个和后一个节点
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

**转换为红黑树的条件：**

1. 当链表上的元素个数超过 8 个

   ```java
   // 判断链表长度到达8调用 treeifyBin 方法转换红黑树
   // 其中TREEIFY_THRESHOLD的值为8，如下：
   // static final int TREEIFY_THRESHOLD = 8;
   if (binCount >= TREEIFY_THRESHOLD - 1)
       treeifyBin(tab, hash);
   ```

2. 但是只有在数组长度 >= MIN_TREEIFY_CAPACITY 时才会转化成红黑树；

   ```java
   // treeifyBin方法开头有判断数组长度是否小于64,小于则进行扩容,否则转红黑树.
   // 其中:MIN_TREEIFY_CAPACITY的值为64.
   // static final int MIN_TREEIFY_CAPACITY = 64;
   final void treeifyBin(Node<K,V>[] tab, int hash) {
       int n, index; Node<K,V> e;
       if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
           resize();
       ...
   }
   ```

   > 参考：https://blog.csdn.net/g5zhu5896/article/details/82968287

当节点变成树节点，以提高搜索效率和插入效率到 O(logN)。

## 构造函数

HashMap 有四个构造函数，如下：

```java
// 给定了初始容量与负载因子的构造函数
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    // 对tableSizeFor进一步分析，这里复制给了threshold，而在第一次调用resize()时，会将threshold值作为容器的初始值
    this.threshold = tableSizeFor(initialCapacity);
}

// 只给了初始容量的，则负载因子为默认的DEFAULT_LOAD_FACTOR=0.75
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 都是默认的情况
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

// 用一个map构造另一个map
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

从上面的代码可以看出，**在常规构造器中，没有为数组table分配内存空间**（除了有一个入参为指定Map的构造器例外），**而是在第一次执行put操作的时候才真正构建table数组**(在`resize()` 函数中会对此过程进行分析)

> ```java
> // 对tableSizeFor进一步分析
> static final int tableSizeFor(int cap) {
>        // 一系列的右移+|运算
>        int n = cap - 1;
>        n |= n >>> 1;
>        n |= n >>> 2;
>        n |= n >>> 4;
>        n |= n >>> 8;
>        n |= n >>> 16;
>        // 最终返回的结果+1，正好是大于等于给定容量的2的幂次方数
>        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
> }
> 
> // 假设初始容量设置为10，后续进行分析
> int n = cap - 1;  // n = 10-1 = 9
>     0000 0000 0000 0000 0000 0000 0000 1001  //9
> n |= n >>> 1;     // n右移1位与n进行或操作
>        0000 0000 0000 0000 0000 0000 0000 0100  //4  n >>> 1
> |   0000 0000 0000 0000 0000 0000 0000 1001  //9 
> -----------------------------------------------------
> 	0000 0000 0000 0000 0000 0000 0000 1101  //13    
> n |= n >>> 2;   // n右移2位与n进行或操作
> 	0000 0000 0000 0000 0000 0000 0000 0011  //3  n >>> 2
> |	0000 0000 0000 0000 0000 0000 0000 1101  //13  
> ---------------------------------------------------
> 	0000 0000 0000 0000 0000 0000 0000 1111  //15 
> n |= n >>> 4;   // n右移4位与n进行或操作
> 	0000 0000 0000 0000 0000 0000 0000 0000  //0
> |	0000 0000 0000 0000 0000 0000 0000 1111  //15
> ----------------------------------------------------
> 	0000 0000 0000 0000 0000 0000 0000 1111  //15
> // 后面的结果都是一样的
> ```
>
> 总结：通过一系列**右移+或运算**后，能够将初始值减一得到的值，后面的所有0变成1，最终返回的是得到的值+1的结果作为容量，**正好就是大于等于给定容量的2的幂次方数**

## HashMap 的 put&get 执行过程

### put 执行过程

**源代码**：

```java
public V put(K key, V value) {
    // 1. 通过hash(key)计算key的hash值
    // 2. 拿到了 hash 值后，调用 putVal()
    return putVal(hash(key), key, value, false, true);
}

// 1. 通过hash(key)计算key的hash值，
static final int hash(Object key) {
    int h;
    // 扰动避免算法，充分利用高低位
    // 其中：>>> 无符号右移，即高位补0
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

// 2. 拿到了hash值后，调用putVal()
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // 定义辅助变量
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 2.1 判断table是否为空
    // hashmap对象中，若tabel属性为空->第一次put->resize()
    if ((tab = table) == null || (n = tab.length) == 0)
        // 调用resize函数，默认大小为16，对于resize函数后续分析☆
        n = (tab = resize()).length;
    // 2.2 发现tab[i]没有值，直接存入即可
    // 根据key的hash值计算该key在hashmap中的位置
    if ((p = tab[i = (n - 1) & hash]) == null)
        // 发现tab[i]没有值，则将元素放入table中
        tab[i] = newNode(hash, key, value, null);
    else {
        // 2.3 tab[i]取到值了，莫慌，先定义下方2个变量
        Node<K,V> e; K k;
        // 2.3.1 如果是key重复了，很简单，直接e = p，即value替换一下即可
        if (p.hash == hash &&  // 当前索引位置对应链表的头节点的hash值和待添加元素值相同
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 2.3.2 发生了hash冲突
        // 2.3.2.1 该链为树，则将该节点插入树中
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 2.3.2.2 该链为链表，则插入到链表中
        else {
            // 开始遍历链表
            for (int binCount = 0; ; ++binCount) {
                // 节点后已经没元素了，则插入
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 添加完成后，判断是需要树化
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 遍历过程中判断节点key和待插入节点是否相同，满足则说明有重复key
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // e不为空，说明产生了hash冲突，且存在了值的替换
        if (e != null) { // existing mapping for key
            V oldValue = e.value;  // 旧的value值返回
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;  // 新的值替换旧的值
            afterNodeAccess(e);  // 空方法，用于后续重写
            return oldValue;
        }
    }
    ++modCount;
    // 是否需要进行扩容
    if (++size > threshold)
        // 若需要扩容，则进行扩容操作，后续分析
        resize();
    afterNodeInsertion(evict);  // 空方法，用于后续重写
    return null;
}
```

**文字说明：**

1. 当我们想往一个 HashMap 中添加一对 key-value 时，系统**首先会计算 key 的 hash 值**；

   > hash 的计算公式：`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)`，后续根据 `hash & (数组长度-1)` 计算在 hash表 中的位置

2. 拿到了 hash 值后，调用 `putVal(...)`，做了如下操作：

   1. 判断 table 是否为空，若为空，则调用 `resize()` 对 table 数组进行初始化；
   2. 根据 hash 值确认在 table 中存储的位置。若该位置没有元素，则直接插入；
   3. 若不为空，存在两种情况：
      1. 如果两个 hash 值相等且 key 值相等：则用新的 Entry 的 value 覆盖原来节点的 value；
      2. 发生了哈希冲突，再次分情况进行讨论：
         1. 若该链已经为树，则将该节点插入树中；
         2. 若该节点为链表，则插入到链表中，而当插入新节点后，若链表的长度过长(大于等于TREEIFY_THRESHOLD)，则将链表转化为红黑树结构。

> 结合上述对 get 分析的结果，给出一个问题：
>
> **HashMap 的 get 方法能否判断某个元素是否在 map 中？**
>
> HashMap 的 get 函数的返回值**不能**判断一个 key 是否包含在 map 中，因为 get 返回 null 有可能是不包含该 key，也有可能该 key 对应的 value 为 null。
>
> **因为 HashMap 中允许 key 为 null，也允许 value 为 null。**
>
> * 测试代码：
>
> ```java
> @Test
> public void test03(){
> 
>     HashMap<String, String> hashMap = new HashMap<>();
>     hashMap.put(null, null);
>     hashMap.put("123", "123");
>     hashMap.put("456", null);
> 
>     System.out.println("size:" + hashMap.size());  // 3
>     System.out.println("key:null, value:" + hashMap.get(null));  // null
>     System.out.println("key:123, value:" + hashMap.get("123"));  // 123
>     System.out.println("key:456, value:" + hashMap.get("456"));  // null
>     System.out.println("key:789, value:" + hashMap.get("789"));  // null
> }
> ```
>
> > 参考：https://mp.weixin.qq.com/s/q1r9Pno6ANUzZ9wMzA-JSg

### get 执行过程

**源代码**：

```java
public V get(Object key) {
    Node<K,V> e;
    // 根据key找value，传入hash(key) 和 key
    // 根据返回结果，为null则直接返回，不然就返回e的value
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 如果表不是空的，并且要查找索引处有值
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 就判断位于第一个的key是否是要查找的key，即hash值相等的情况下，内容也要相同
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 说明第一个结点的值不相同，若结点后有链表/树继续遍历查找
        if ((e = first.next) != null) {
            // 如果不是就判断链表是否是红黑二叉树，如果是，就从树中取值
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 如果不是树，就遍历链表
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

**文字说明：**

通过 key 的 hash 值找到在 table 数组中的索引处的 Node，然后返回该 key 对应的 value 即可。

在这里能够根据 key 快速的取到 value 除了和 HashMap 的数据结构密不可分外，还和 Node 有莫大的关系。

HashMap 在存储过程中并没有将 key，value 分开来存储，而是当做一个整体 key-value 来处理的，这个整体就是 Node 对象。

同时 value 也只相当于 key 的附属而已。

**即在存储的过程中，系统根据 key 的 HashCode 来决定 Node 在 table 数组中的存储位置`hash(key) & (数组长度-1)`），在取的过程中同样根据 key 的 HashCode 取出相对应的 Node 对象（value 就包含在里面）。**

### `putTreeVal()` 方法

当首节点已经树结点时，则调用该方法，将结点插入到树中

```java
// 当添加元素时，对应的索引位置为TreeNode节点，会执行该方法
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                               int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    TreeNode<K,V> root = (parent != null) ? root() : this;
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;
        if ((ph = p.hash) > h)
            dir = -1;
        else if (ph < h)
            dir = 1;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if ((kc == null &&
                  (kc = comparableClassFor(k)) == null) ||
                 (dir = compareComparables(kc, k, pk)) == 0) {
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null &&
                     (q = ch.find(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                     (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            dir = tieBreakOrder(k, pk);
        }
		// 以上逻辑，就是在遍历红黑树，决定新元素放置的位置
        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            Node<K,V> xpn = xp.next;
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            // 找到新元素放置的位置，并将其添加进红黑树结构
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            // 同时维护双向链表结构
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            // 通过变色+旋转达到红黑树的自平衡
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```

### `treeifyBin()` 方法

在`put()`函数中，当添加元素后，单向链表长度达到8个会执行该方法

```java
// 树化条件1：单向链表长度达到8个
if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
    // 进行树化操作
    treeifyBin(tab, hash);

// 后续对treeifyBin方法做进一步分析
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 树化的条件2：还需要满足此时数组长度 >= 64
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    // 获取第一个hash桶的第一个结点，开始树化操作，这里其实是进行了节点的装环并维护出一个双向链表来
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 把所有Node节点，转成TreeNode节点，并形成双向链表
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                // 首节点
                hd = p;
            else {
                // 双向链表的形成
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            // 将双向链表中的元素形成红黑树结构，这里才是真正的变成红黑树
            hd.treeify(tab);
    }
}
```

进一步：**为什么在维护红黑树的同时，需要再维护一种双向链表的结构呢？其实主要是为了扩容方便的**，进后续扩容操作分析

## HashMap 的 resize/扩容

> 首先说明：**JDK 7 的扩容，是先扩容再添加值**；而**JDK 8的话是先添加值再进行扩容操作**，下面是对于 JDK 8 的扩容源码分析，至于 JDK7 会在下一节多线程安全问题中涉及

**有两种情况会调用 resize 方法：**

### 情况1

第一次调用 HashMap 的 **put 方法**时，会调用 resize 方法对 table 数组进行初始化，如果不传入指定值，默认大小为 16。

```java
// hashmap对象中 tabel属性为空--->第一次put---->resize()
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
```

### 情况2

HashMap 中的容量超过了阈值(threshold)，会进行扩容；

在 `put()` 方法中成功将新节点放入hash表会进行检测，有如下：

```java
if (++size > threshold)
    resize();
```

其中：`threshold=loadFactor * length`，也就是说数组长度固定以后， 如果负载因子越大，所能容纳的元素个数越多，如果超过这个值就会进行扩容(**默认是扩容为原来的2倍**)

> 如果在一些特殊情况下，比如空间比较多，但要求速度比较快，这时候就可以把扩容因子调小以较少hash冲突的概率
>
> 相反就增大扩容因子(这个值可以大于1的)

### resize 详解

**resize 源码**：

```java
final Node<K,V>[] resize() {
	// table原容器
    Node<K,V>[] oldTab = table;
    // 判断是否是初始化，若是初始化的话，则这个oldCap为0
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 当原容器的size已经是最大了，则不进行扩容，直接返回
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 计算新容器的长度，为原来的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
			// 阈值也同时变了两倍
            newThr = oldThr << 1; // double threshold
    }
    // 初始化容器，调用的构造为传入初值容量的，由于函数tableSizeFor存在，newCap必为2的幂次方数
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;  // 然后newThr会在后续进行再次计算
    // 初始化容器，调用的构造为不传入初始容量的，或者传入的是0
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 创建新的数组
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 真正开始扩容操作，若是初始化不会进入这里
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            // 数据角标位置有值，则需要进行迁移
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 数组角标位置只有一个元素，直接将数据迁移到新数组
                if (e.next == null)
                    // 直接通过e.hash & (newCap - 1)计算新的位置
                    newTab[e.hash & (newCap - 1)] = e;
                // 数组角标位置为TreeNode，迁移红黑树数据
                else if (e instanceof TreeNode)
                    // 树节点的迁移：后续分析
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    // 迁移单向链表数据，这里其实是分了两个链表，高低两个链表
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 先遍历整个单向链表，元素放置的位置：
                    //   1. 要么是原来的位置; 2. 要么是原来位置+扩容容量的位置
                    do {
                        next = e.next;
                        // 放置在原来角标位置的元素
                        // 这个 oldCap 就是 10..00的值，根据这个最高位是0还是1来决定放到原位置/+扩容容量位置
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 放置在原来角标+扩容容量 位置的元素
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 将放置在原角标位的元素存入新数组
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 将放置在新角标位的元素存入新数组
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

**文字说明**：

每次扩容之后容量都是**翻倍**。扩容后要将原数组中的所有元素找到在新数组中合适的位置。

有如下：当我们把 `table[i]` 位置的所有 `node` 迁移到 `newtab` 中去的时候：这里面的 `node` 要么在 `newtab` 的 `i` 位置（不变），要么在 `newtab` 的 `i + n` 位置。

> 其中 n 为 原table 的长度，代码中为 `oldCap`

也就是我们可以这样处理：把 `table[i]` 这个桶中的 `node` 拆分为两个链表 L1 和 L2：如果 `hash & n == 0`，那么当前这个 `node` 被连接到 L1 链表；否则连接到 L2 链表。这样下来，当遍历完 `table[i]` 处的所有 node 的时候，我们得到两个链表 L1 和 L2，这时我们令 `newtab[i] = L1`，`newtab[i + n] = L2`，这就完成了 `table[i]` 位置所有 node 的迁移（rehash），**这也是 HashMap 中容量一定的是 2 的整数次幂带来的方便之处**。

> 以下内容摘录自博文：[HashMap JDK1.8实现原理：扩容机制](https://www.cnblogs.com/duodushuduokanbao/p/9492952.html)，讲的挺好，摘录一下，不然以后找不到了:dog2:
>
> 我们使用的是 **2次幂** 的扩展(指长度扩为原来2倍)，所以，**元素的位置要么是在原位置，要么是在原位置再移动 2次幂 的位置**。
>
> 看下图可以明白这句话的意思，n为table的长度，图(a)表示扩容前的 key1 和 key2 两种 key 确定索引位置的示例，图(b)表示扩容后 key1 和 key2 两种 key 确定索引位置的示例；
>
> > 其中 hash1 是 key1 对应的哈希与高位运算结果。
>
> ![HashMap扩容1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251614913.png)
>
> 元素在重新计算 hash 之后，因为 n 变为 2 倍，那么 n-1 的 mask 范围在高位多1bit(红色)，因此新的 index 就会发生这样的变化：
>
> ![HashMap扩容2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251614568.png)
>
> 因此，我们在扩充 HashMap 的时候，不需要像 JDK1.7 的实现那样重新计算 hash，只需要看看原来的 hash 值新增的那个 bit 是1还是0就好了，是0的话索引没变，是1的话索引变成**“原索引+oldCap”**，可以看看下图为16扩充为32的resize示意图：
>
> ![HashMap扩容3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251614839.png)
>
> 这个设计确实非常的巧妙，既省去了重新计算 hash 值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此 resize 的过程，均匀的把之前的冲突的节点分散到新的 bucket 了。这一块就是 JDK1.8 新增的优化点。有一点注意区别，JDK1.7中 rehash 的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置，但是从上图可以看出，**JDK1.8不会倒置**。

结合上述：HashMap 的 size 为什么必须是 2 的整数次方

### resize 为 2的幂次

> 结合上述 resize 过程理解

首先有：HashMap 的 size 必须是 2 的整数次方

**总的来讲：是为了将 key 的 hash 值均匀的分布在数组的索引上，减少哈希冲突的出现。**

1. 这样做总是能够保证 HashMap 的底层数组长度为 2 的 n 次方。当 length 为 2 的 n 次方时，`hash(key)&(length - 1)`  就相当于对 length 取模 `(hash(key)%length)`，而且速度比直接取模快得多，**这是 HashMap 在速度上的一个优化**。
2. 如果 length 为 2 的次幂，则 length - 1 转化为二进制必定是 `11111……` 的形式，在与 key 的hash值进行二进制进行与操作时效率会非常的快，而且**空间不浪费**。
3. 如果 length 不是 2 的次幂，比如：length 为 15，则 length - 1 为 14，对应的二进制为 `1110`，在于 key的hash 与操作，最后一位都为 0 ，而 `0001，0011，0101，1001，1011，0111，1101` 这几个位置永远都不能存放元素了，**空间浪费相当大**，更糟的是这种情况中，数组可以使用的位置比数组长度小了很多，这意味着进一步增加了碰撞的几率，减慢了查询的效率，这样就会造成空间的浪费。
4. **便于扩容**：扩充 HashMap 的时候，不需重新计算 hash，只需要看看原来的 hash 值新增的那个 bit 是1还是0就好了，是0的话索引没变，是1的话索引变成**“原索引+oldCap”**，更为具体的可见上一节：**resize 详解**

### 红黑树数据迁移 split

上述在分析 resize 函数时，只是对普通的 Node 节点如何迁移进行了分析，对于其中已经转换为 TreeNode 的树节点如何迁移未做分析，此处进行进一步分析：

```java
// resize方法中通过了TreeNode中的方法split对红黑树节点数据进行了迁移
// 此处的e节点当然是hash在相应索引位置的第一个元素，即为红黑树的根节点
((TreeNode<K,V>)e).split(this, newTab, j, oldCap);

// 传入参数：1.HashMap类；2.扩容后的hashmap；3.原索引位置；4.原数组长度
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    TreeNode<K,V> b = this;
    // Relink into lo and hi lists, preserving order
    // 这里也分成了两个链表
    TreeNode<K,V> loHead = null, loTail = null;
    TreeNode<K,V> hiHead = null, hiTail = null;
    int lc = 0, hc = 0;
    // 通过遍历双向链表，实现数据迁移，这里的逻辑其实和单向链表的节点迁移大致相同
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        e.next = null;
        // 原角标位置
        if ((e.hash & bit) == 0) {
            // 记录前一个
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                // 记录下一个 -- 尾插法
                loTail.next = e;
            loTail = e;
            // 该标记累计，用于判断是否需要转会单向链表
            ++lc;
        }
        // 原角标+扩容容量 位置
        else {
            // 记录前一个
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                // 记录下一个 -- 尾插法
                hiTail.next = e;
            hiTail = e;
            // 该标记累计，用于判断是否需要转会单向链表
            ++hc;
        }
    }

    if (loHead != null) {
        // 元素个数小于等于6，转成单向链表，仅在这里做了非树化操作！
        if (lc <= UNTREEIFY_THRESHOLD)
            tab[index] = loHead.untreeify(map);
        else {
            // 存入新的数组
            tab[index] = loHead;
            if (hiHead != null) // (else is already treeified)
                // 树化
                loHead.treeify(tab);
        }
    }
    if (hiHead != null) {
        // 元素个数小于等于6，转成单向链表
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            // 存入新的数组
            tab[index + bit] = hiHead;
            if (loHead != null)
                // 树化
                hiHead.treeify(tab);
        }
    }
}
```

## HashMap 多线程下的问题

主要是在扩容时会存在问题，分 JDK7 和 JDK8 进行分析

### JDK7 死循环

主要是多线程同时 put 时，如果同时触发了 resize 操作，会导致 HashMap 中的链表中出现**循环节点**，进而使得后面 get 的时候，会死循环。

通过查看 JDK7 `put()` 、`addEntry()`、`resize()`、`transfer()`函数说明死循环出现过程

```java
public V put(K key, V value) {
    // HashMap允许存储null键，存储在数组的0索引位置
    if (key == null)
        // JDK7中通过 putForNullKey 函数来处理key为null
        return putForNullKey(value);
    // 内部通过一个扰乱算法获得一个hash值，用于计算数组索引
    int hash = hash(key);
    // 计算数组索引，找到那个hash桶的位置
    int i = indexFor(hash, table.length);
    // 判断是否是重复键
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    // 添加元素
    addEntry(hash, key, value, i);
    return null;
}

// 添加新节点
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 元素个数大于阈值，同时当前索引位有值，就会执行扩容操作
    if ((size >= threshold) && (null != table[bucketIndex])) {
        // 2倍扩容，这里就会出现线程安全问题！！！
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        // 重新计算索引位置
        bucketIndex = indexFor(hash, table.length);
    }
	// 基于键值创建Entry节点，并以头插法存入对应位置，在扩容之后再添加数据
    createEntry(hash, key, value, bucketIndex);
}

// 需要经过扩容
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    // 达到最大容量，不扩容
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    // 根据新容量创建数组
    Entry[] newTable = new Entry[newCapacity];
    boolean oldAltHashing = useAltHashing;
    useAltHashing |= sun.misc.VM.isBooted() &&
            (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    // 计算是否需要重新计算hash
    boolean rehash = oldAltHashing ^ useAltHashing;
    // 将旧数组中的元素迁移到新的数组中
    transfer(newTable, rehash);
    // 保存新数组
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

// 扩容过程中当然涉及到了节点的转移操作
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    // 遍历数据的每一个角标位
    for (Entry<K,V> e : table) {
        // 当数组对应的桶位置不为null，则需要进行数据迁移
        while(null != e) {
            // 记录遍历到的元素的下一个元素
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            // 计算新数组的角标位置，有两种情况：在原先位置 / 原位置+原长度
            int i = indexFor(e.hash, newCapacity);
            // 把当前元素的下一个改为新数组对应位置的元素--头插法
            e.next = newTable[i];
            // 将当前元素放置在数组对应索引位置
            newTable[i] = e;
            // 再次迁移下一个元素
            e = next;
        }
    }
}
```

**结合上述代码进行文字+图片说明**：

1. 假定 thread-1, thread-2 都进入了 `transfer()` 函数进行扩容时的数据迁移工作，假定此时的 table 结构如下：

   ![resize造成死循环1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251614651.PNG)

2. 当 thread-1 执行完 `Entry<K,V> next = e.next;`后失去的 CPU 的使用权，现在 thread-1 的状态就是上面的状态；

3. thread-2 获取 CPU 的执行权，也进入这里执行，起始状态也是上图所示；

4. thread-2 执行后续逻辑完成扩容，最终结果如下（头插法）：

   ![resize造成死循环2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251614910.PNG)

5. thead-1 再次获取CPU的执行权，此时 e 和 next 的节点指向恢复，然后继续到 `e.next = newTable[i];`

   ![resize造成死循环3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251614346.PNG)

6. 当 thread-1 执行完 `e.next = newTable[i];`后存在如下结构，即存在了环状结构

   ![resize造成死循环4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251614790.PNG)

### JDK8 数据丢失

而 JDK8 中已经不使用头插法了，而是使用的尾插法，但是在多线程情况下，也会存在数据丢失的问题

```java
// 添加元素时，会有数据覆盖丢失数据
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 此处，如果多个线程向同一个位置存入元素，会有值覆盖的问题，导致数丢失
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

    // 下面代码省略
}

// resize函数中：扩容时，迁移数据的情况下，会有数据覆盖丢失的问题
// 多线程环境下，给同一个数组的相同位置赋值，会有数据覆盖的风险
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;  // 将原始索引位的数据迁移到新数组
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead; // 将新索引位的数据迁移到新数组
}
```

## 参考

https://www.cnblogs.com/chengxiao/p/6059914.html

https://blog.csdn.net/vking_wang/article/details/14166593

https://blog.csdn.net/zjcjava/article/details/78495416

https://www.cnblogs.com/zhimingxin/p/8609545.html

https://www.cnblogs.com/kangkaii/p/8473793.html

https://space.bilibili.com/488677881/video