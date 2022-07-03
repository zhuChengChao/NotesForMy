# Java：LinkedHashMap类

> 对 Java 中的 **LinkedHashMap类**，做一个微不足道的小小小小记

## 概述

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>  // 继承于 HashMap
    implements Map<K,V>{
	// ...
}
```

LinkedHashMap 继承于 HashMap，而由于 HashMap 是无序的，而当我们需要有序的存储 key-value 键值对时，就可以采用 LinkedHashMap。

> 关于 HashMap 的进一步了解：[Java：HashMap类小记](https://www.cnblogs.com/zhuchengchao/p/14298141.html)

**因此，LinkedHashMap其实就是可以看成 HashMap 的基础上，多了一个双向链表来维持顺序。**

下面用一张图来表示：

![LinkedHashMap](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251615633.JPG)

## 实现原理

由于 LinkedHashMap 继承于 HashMap，因此很多都是直接对原 HashMap 函数进行了些许的重写处理

### 成员属性

LinkedHashMap 也是基于 HashMap 实现的，不同的是它定义了一个 **Entry head，tail**，这个 head,tail 不是放在 Table 里，它是额外独立出来的。LinkedHashMap 通过继承 hashMap 中的 Node，并添加两个属性 **Entry before，Entry after** 并结合 head，tail组成一个双向链表，来**实现按插入顺序或访问顺序排序**。

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>{
    // LinkedHashMap实现了静态内部类Entry，继承了HashMap.Node，
    // 同时定义了before, after两个属性，保证了有序性
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
	// 链表的头结点，表示最老的那个节点
    transient LinkedHashMap.Entry<K,V> head;
	// 链表的尾节点，表示最新插入的那个节点
    transient LinkedHashMap.Entry<K,V> tail;
	// 链表的遍历顺序：
    final boolean accessOrder;
}
```

### 构造函数

从下面的构造函数可以看出，LinkedHashMap 都是调用了 HashMap 的构造函数，只是多了一个 **accessOrder** 的成员属性

> accessOrder 与存储的顺序有关，LinkedHashMap 存储数据是有序的，而且分为两种：**插入顺序**和**访问顺序**，见后续分析

```java
// 考虑到LinkedHashMap继承自HashMap，因此super指的即为HashMap的构造函数
public LinkedHashMap(int initialCapacity, float loadFactor) {
    // 调用HashMap的构造函数
    super(initialCapacity, loadFactor);
    // false 默认是插入顺序
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

public LinkedHashMap() {
    super();
    accessOrder = false;
}

public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}

public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    // 通过指定accessOrder来控制插入/访问顺序
    this.accessOrder = accessOrder;
}
```

### put 方法

在 JDK1.8 的 LinkedHashMap 源码中，甚至没有 `put()` 方法，因此它继承的是来自 HashMap 的 put 方法

> 对于涉及到HashMap的一些方法，在[Java：HashMap类小记](https://www.cnblogs.com/zhuchengchao/p/14298141.html)中已经分析了，这里也就不再分析，**只分析 LinkedHashMap 对其重写的方法**

```java
// HashMap的put方法
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        // 不同处1.newNode在LinkedHashMap中对其进行了重写
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    // 不同处1.newNode在LinkedHashMap中对其进行了重写，后续进行分析
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 不同处2.在HashMap中为空方法，在LinkedHashMap进行了实现，后续进行分析
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    // 不同处3.在HashMap中为空方法，在LinkedHashMap进行了实现
    afterNodeInsertion(evict);
    return null;
}

// 1.newNode在LinkedHashMap中对其进行了重写
// 实现了：LinkedHashMap创建Entry，通过linkNodeLast方法将Entry接在双向链表的尾部，实现了双向链表的建立
// Create a regular (non-tree) node
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    // 将Entry接在双向表尾部
    linkNodeLast(p);
    return p;
}

// link at the end of list
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    // p为待插入节点，则p为新的tail节点
    tail = p;
    if (last == null)
        // 只有一个节点插入的情况
        head = p;
    else {
        // p的前一个节点为原先的tail节点
        p.before = last;
        // 原先的tail节点的after为p节点
        last.after = p;
    }
}
```

### remove 方法

同样，在 JDK1.8 的 LinkedHashMap 源码中，甚至没有 `remove()` 方法，因此它继承的是来自 HashMap 的 `remove()` 方法

```java
// HashMap的remove方法
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
	
    // ...
    
    // 这个方法在LinkedHashMap中进行了重写
    afterNodeRemoval(node);
    
    // ...
}

// LinkedHashMap删除节点，调用HashMap的remove方法删除单链表的节点，
// 再重写afterNodeRemoval方法，删除双向链表的节点。
void afterNodeRemoval(Node<K,V> e) { // unlink
    // p为待删除节点，p为待删除节点前一个节点，a为待删除节点后一个节点
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;  // 待删除节点为头结点
    else
        b.after = a;  // 把链表中p节点的位置删除
    if (a == null)
        tail = b;  // 待删除节点为尾节点
    else
        a.before = b;  // 把链表中p节点的位置删除
}
```

### 访问顺序

上文提到：成员变量：**accessOrder** 与存储的顺序有关，LinkedHashMap存储数据是有序的，而且分为两种：**插入顺序**和**访问顺序**

插入顺序：默认，`accessOrder=false`，即按照插入元素的顺序进行排序；

访问顺序：手动设定`accessOrder=true`，按访问顺序排序，即每当访问一次元素，该元素就会被移动链表的最后面

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        // 当指定为访问排序，则每次访问后会调用afterNodeAccess重新排序
        afterNodeAccess(e);
    return e.value;
}

// 通过函数accessOrder，把访问到的节点e放到双向链表的最后，即变为最新节点
// 该函数在put()函数调用时，若存在hash冲突，在更新完节点后也会被调用
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

### 缓存

在 HashMap 中的三个空实现函数，在 LinkedHashMap 中都给出了具体的实现：

* `void afterNodeAccess(Node<K,V> p) { }`：维护了访问顺序
* `void afterNodeInsertion(boolean evict) { }`：维护了插入顺序
* `void afterNodeRemoval(Node<K,V> p) { }`：维护了删除顺序

还差一个 afterNodeInsertion 未分析，该函数在每次添加完元素后调用，源代码如下：

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    // 条件1：evict=true，源码里一直为true的
    // 条件2：即head不为null，即 LinkedHashMap 不为空
    // 条件3：removeEldestEntry 重写函数
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

从上分析，只需重写 `removeEldestEntry` 函数，其实就可以实现一些缓存策略，比如 LRU 等...

```java
class LRUCache {

    int capacity;
    LinkedHashMap<Integer, Integer> cache;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        cache = new LinkedHashMap<Integer, Integer>(capacity, 0.75f, true){	
            // 重写一下方法removeEldestEntry
            @Override
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return cache.size() > capacity;
            }
        };
    }
    
    public int get(int key) {
        return cache.getOrDefault(key, -1);
    }
    
    public void put(int key, int value) {
        cache.put(key, value);
    }
}
```

> 可见：[146. LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)

## 参考：

图解LinkedHashMap原理：https://www.jianshu.com/p/8f4f58b4b8ab

> JDK 1.7 的 LinkedHashMap 实现

LinkedHashMap 源码详细分析：https://segmentfault.com/a/1190000012964859

> JDK 1.8 中的实现

