# Java：TreeMap类

> 对 Java 中的 **TreeMap类**，做一个微不足道的小小小小记

## 概述

前言：之前已经小小分析了一波 [HashMap类](https://www.cnblogs.com/zhuchengchao/p/14298141.html)、[HashTable类](https://www.cnblogs.com/zhuchengchao/p/14444323.html)、[ConcurrentHashMap类](https://www.cnblogs.com/zhuchengchao/p/14446548.html)、[LinkedHashMap类](https://www.cnblogs.com/zhuchengchao/p/14447418.html)，现在再小小分析一下 TreeMap 类

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
    // ...
}
```

TreeMap 的实现就是**红黑树数据结构**，也就说是一棵自平衡的排序二叉树，这样就可以保证当需要快速检索指定节点，根据 key 找节点的时间复杂度在为 O(logn) 级别

TreeMap 中的每个 Entry 都被当成“红黑树”的一个节点对待；

```java
// 红黑树节点定义
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;  // 当前节点的左子树
    Entry<K,V> right; // 当前节点的右子树
    Entry<K,V> parent;  // 当前节点的父节点
    boolean color = BLACK;  // 红/黑
    
    // ... 省略一下构造方法/get/set方法
	
    // 判断节点是否相等
    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;

        return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
    }
	
    // 计算Entry的hash值
    public int hashCode() {
        int keyHash = (key==null ? 0 : key.hashCode());
        int valueHash = (value==null ? 0 : value.hashCode());
        return keyHash ^ valueHash;
    }
}
```

> 需要说明的是，关于红黑树的构建，调整等操作，这些主要涉及的是数据结构方面的知识点，这里不做过多分析。

## 实现原理

### 成员属性

```java
// 比较器：由于TreeMap实现的是红黑树数据结构，而其是通过key进行比较的，从而达到有序的状态
private final Comparator<? super K> comparator;
// 树的根节点
private transient Entry<K,V> root;
// 集合大小
private transient int size = 0;
// 存储结构被修改的次数
private transient int modCount = 0;

// 红黑树节点定义
static final class Entry<K,V> implements Map.Entry<K,V> {
    // 上面已经给出了
}
```

### 构造方法

```java
// 1. 默认构造函数，不传入比较器
// 如果是这样的话，则key是需要实现Comparable接口的，Comparable接口中的compareTo才可以用于比较
public TreeMap() {
    comparator = null;
}

// 2. 自定义比较器的构造方法
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}

// 3. 用已知的Map对象构造TreeMap
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}

// 4. 构造已知的SortedMap对象为TreeMap
public TreeMap(SortedMap<K, ? extends V> m) {
    // 使用已知对象的构造器
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

### put 方法

```java
public V put(K key, V value) {
    // 获取根节点
    Entry<K,V> t = root;
    if (t == null) {
        // 如果根节点为空，则该元素置为根节点，一般为第一次添加 
        // ☆compare☆后续分析
        compare(key, key); // type (and possibly null) check
		// 根节点为null，则令当前节点为root节点
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;  // 比较结果定义
    Entry<K,V> parent;
    // split comparator and comparable paths
    // 获取比较器对象☆
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        // 如果比较器对象不为空，也就是自定义了比较器
        do {
            parent = t;  // t就是root
            // 调用比较器对象的compare()方法，该方法返回一个整数，表示比较结果
            cmp = cpr.compare(key, t.key);  // root的key值和待插入节点的key值进行比较
            if (cmp < 0)
                // 待插入元素的key"小于"当前位置元素的key，则查询左子树
                t = t.left;
            else if (cmp > 0)
                // 待插入元素的key"大于"当前位置元素的key，则查询右子树
                t = t.right;
            else
                // "相等"则替换其value。
                return t.setValue(value);
        } while (t != null);
    }
    else {  // 如果比较器对象为空，使用key的默认比较机制，因此这里的key比较是实现了Comparable接口的
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        // 由于这里key实现了Comparable，因此可以进行强转
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            // 同样是循环比较并确定元素应插入的位置(也就是找到该元素的父节点)
            parent = t;
            // 同样调用比较方法并返回一个整数
            // 用待插入的key去比较，有如下：
            //   <0 说明待插入的比较小，走左子树
            //   >0 说明待插入的比较大，走右子树
            //   =0 说明key大小相同，则替换value
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                // 待插入元素的key"小于"当前位置元素的key，则查询左子树
                t = t.left;
            else if (cmp > 0)
                // 待插入元素的key"大于"当前位置元素的key，则查询右子树
                t = t.right;
            else
                // "相等"则替换其value。
                return t.setValue(value);
        } while (t != null);
    }
    // 根据key找到父节点后新建一个节点
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0) // 根据比较的结果来确定放在左子树还是右子树
        parent.left = e;
    else
        parent.right = e;
    // 由于在插入节点后，红黑树的平衡性会被打破，因此会通过左旋/右旋进行调整的
    fixAfterInsertion(e);
    size++;      // 集合大小+1
    modCount++;  // 集合结构被修改次数+1
    return null;
}
```

> 后续的删除节点其实也是大同小异，也就不看了，推荐一篇博文：
>
> https://blog.csdn.net/jtcode_is_my_partner/article/details/81408392
>
> 里面讲了修复的过程

### compare

使得 TreeMap 能有序的主要原因就是这个比较，把新增节点与 根节点/左子节点/右子节点 不断的比较，直到找到合适的位置进行插入

先给出 **compare** 方法：

```java
private final Comparator<? super K> comparator;

final int compare(Object k1, Object k2) {
    // 当比较器为null时，用key自带的比较器，即key需要实现Comparable接口
    return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
        : comparator.compare((K)k1, (K)k2);
}
```

**案例1**：使用默认的比较器

```java
TreeMap<String, Integer> treeMap = new TreeMap<>();

treeMap.put("aaa", 1);
treeMap.put("bbb", 2);
treeMap.put("ddd", 4);
treeMap.put("ccc", 3);

// 可见默认的就是string的comparator
Comparator<? super String> comparator = treeMap.comparator();
Set<Map.Entry<String, Integer>> entries = treeMap.entrySet();
for (Map.Entry<String, Integer> entry: entries){
    System.out.println(entry.getKey() +":"+ entry.getValue());
}

// 输出：
// aaa:1
// bbb:2
// ccc:3
// ddd:4
```

**案例2**：自定义比较器

> 关于比较器的定义方式，可见：[Java：常用的容器小记：Comparable 与 Comparator 两个接口的区别](https://www.cnblogs.com/zhuchengchao/p/14296844.html)

```java
// 升序：
TreeMap<Integer, String> treeMap = new TreeMap<>(new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1-o2;
    }
});

treeMap.put(1, "aaa");  // 输出顺序：1
treeMap.put(2, "bbb");  // 输出顺序：2
treeMap.put(4, "ddd");  // 输出顺序：3
treeMap.put(3, "ccc");  // 输出顺序：4

Set<Map.Entry<Integer, String>> entries = treeMap.entrySet();
for (Map.Entry<Integer, String> entry: entries){
    System.out.println(entry.getKey() +":"+ entry.getValue());
}

// 降序
TreeMap<Integer, String> treeMap = new TreeMap<>((o1, o2) -> o2-o1);

treeMap.put(1, "aaa");  // 输出顺序：4
treeMap.put(2, "bbb");  // 输出顺序：3
treeMap.put(4, "ddd");  // 输出顺序：2
treeMap.put(3, "ccc");  // 输出顺序：1

// 获取比较器
Comparator<? super Integer> comparator = treeMap.comparator();
Set<Map.Entry<Integer, String>> entries = treeMap.entrySet();
for (Map.Entry<Integer, String> entry: entries){
    System.out.println(entry.getKey() +":"+ entry.getValue());
}
```

### get 方法

**根据key获取元素**

```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    if (comparator != null)
        // 如果有自定义比较器对象，就按照自定义规则遍历二叉树，见下述函数
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    
    // 没有自定义比较器，则按照key的默认比较规则进行查找
    @SuppressWarnings("unchecked")
    Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    while (p != null) {
        // 按照对于比较规则遍历二叉树
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}

final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
    K k = (K) key;
    // 获取比较器
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        Entry<K,V> p = root;
        while (p != null) {
            // 进行比较找到相应的节点
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                // 待比较key小于当前位置元素的key，迭代查询左子树
                p = p.left;
            else if (cmp > 0)
                // 待比较key大于当前位置元素的key，迭代查询右子树
                p = p.right;
            else
                return p;
        }
    }
    return null;
}

// 获取第一个元素/最小的元素
public Map.Entry<K,V> firstEntry() {
    return exportEntry(getFirstEntry());
}
// 根据二叉搜索树的性质，最左边的节点为最小的节点
final Entry<K,V> getFirstEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.left != null)
            p = p.left;
    return p;
}
```

## 参考

https://blog.csdn.net/jtcode_is_my_partner/article/details/81408392

https://blog.csdn.net/qq_32166627/article/details/72773293