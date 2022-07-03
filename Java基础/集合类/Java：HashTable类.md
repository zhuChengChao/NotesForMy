# Java：HashTable类

> 对 Java 中的 **HashTable类**，做一个微不足道的小小小小记

## 概述

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    // ...
}
```

需要说明的是：HashTable 和 HashMap 的实现原理基本一样，**差别**无非是：

1. HashTable 不允许 key 和 value 为 null；

2. HashTable 是线程安全的。

   但是 HashTable 线程安全的策略实现代价却太大了，简单粗暴，**get/put 所有相关操作都是 synchronized 的**，这相当于给整个哈希表加了一把大锁，多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞，相当于将所有的操作串行化，在竞争激烈的并发场景中性能就会非常差。

3. 扩容方式稍有不同，见后续分析

> 对于 HashMap 的一点记录，见：[Java：HashMap类小记](https://www.cnblogs.com/zhuchengchao/p/14298141.html)

## 相同点

对比了 HashTable 与 HashMap 的相同点，如下：

1. 都可以用来存储键值对
2. 底层哈希表结构查询速度都很快
3. 内部通过单链表解决冲突问题，容量不足会自动增加
4. 都实现了 Map 接口
5. 都实现了 Serializable 接口，支持序列化
6. 实现了 Cloneable 接口，可以被克隆

## 不同点

### 继承体系

HashTable 继承了 **Dictionary 类**，而 HashMap 是继承了 **AbstractMap类**。Dictionary 是任何可将键映射到相应值的类的抽象父类，而 AbstractMap 是基于 Map 接口的实现，它以最大限度地减少实现此接口所需的工作。

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
	...
}
public abstract class AbstractMap<K,V> implements Map<K,V> {
	...
}

/**********************分割线**************************/

public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    ...
}

public abstract class Dictionary<K,V> {
    ...
}
```

> 据说这是因为：历史原因

### 线程安全

**线程安全不一样：**Hashtable 是线程安全的，而 HashMap 不是线程安全的，但是我们也可以通过 `Collections.synchronizedMap(hashMap)`，使其实现同步。

```java
// 这是Hashtable的put()方法:
public synchronized V put(K key, V value){
    ...
}

/**********************分割线**************************/

// 这是HashMap的put()方法:
public V put(K key, V value) {
    ...
}
```

从上面的源代码可以看到 Hashtable 的 `put()` 方法是**synchronized**的，而 HashMap 的 `put()` 方法却不是。

### 存放内容

**HashMap 的 key 和 value 都允许为 null，而 Hashtable 的 key 和 value 都不允许为 null。**

HashMap 遇到 key 为 null 的时候，调用 putForNullKey 方法进行处理（JDK 8中在hash函数中做了判断），而对 value 没有处理，value就可以为null。

Hashtable 遇到 null，直接返回 `NullPointerException`。

```java
// HashMap在put值时，在计算key的hash值时有null的判断
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 在 hash(key)方法中：对于null做了特殊的处理
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

/**********************分割线**************************/

// Hashtable
public synchronized V put(K key, V value) {
    // Make sure the value is not null 
    // 1.值为null直接抛出异常
    if (value == null) {
        throw new NullPointerException();
    }
    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    // 由于key是null，null当然是没有hashCode()这种方法的
    // 因此上述需要直接抛出异常NullPointerException
    int hash = key.hashCode();  
    int index = (hash & 0x7FFFFFFF) % tab.length;
    // ....
}
```

### 遍历方式

**遍历方式的内部实现上不同**：HashMap 使用 Iterator 遍历，HashTable 使用Enumeration 遍历；

```java
// HashMap中定义了抽象类HashIterator
// KeyIterator/ValueIterator/EntryIterator都对HashIterator进行了继承，并实现了对应的Iterator接口
abstract class HashIterator {}

final class KeyIterator extends HashIterator
    implements Iterator<K> {
    public final K next() { return nextNode().key; }
}

final class ValueIterator extends HashIterator
    implements Iterator<V> {
    public final V next() { return nextNode().value; }
}

final class EntryIterator extends HashIterator
    implements Iterator<Map.Entry<K,V>> {
    public final Map.Entry<K,V> next() { return nextNode(); }
}

/**********************分割线**************************/

// Hashtable：获取 Enumeration 用于遍历
private <T> Enumeration<T> getEnumeration(int type) {
    if (count == 0) {
        return Collections.emptyEnumeration();
    } else {
        return new Enumerator<>(type, false);
    }
}

private <T> Iterator<T> getIterator(int type) {
    if (count == 0) {
        return Collections.emptyIterator();
    } else {
        return new Enumerator<>(type, true);
    }
}
```

> 据说这也是因为：历史原因
>
> 关于 Iterator 与 Enumeration 的异同，见：[Java：Iterator接口与fail-fast小记](https://www.cnblogs.com/zhuchengchao/p/14297051.html)

### 初始化 & 扩容

HashMap 和 HashTable 的**初始化方式**和**扩容方式**不同：

HashMap：在构造函数中**不创建数组**，而是在第一次put时才创建，且初始大小为16；之后每次扩充容量为原来的两倍；

**HashTable：在构造函数直接创建hashtable，且其初始大小为11，之后每次扩充2n+1**

* HashMap的初始化与扩容（见：[Java：HashMap类小记](https://www.cnblogs.com/zhuchengchao/p/14298141.html)）

```java
// hashmap初始化:HashMap hashMap = new HashMap();
// 默认构造函数
public HashMap() {
    // 先不创建，在使用的时候再创建
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    // 其中：DEFAULT_LOAD_FACTOR = 0.75f;
    // DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 并不创建真正的数组，而是等到第一次put时才通过resize去创建
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
    this.threshold = tableSizeFor(initialCapacity);
}
```

**HashTable 的初始化与扩容**

* **初始化**

```java
// hashtable初始化:Hashtable hashtable = new Hashtable();
public Hashtable() {
    // 构造函数直接创建hashtable
    this(11, 0.75f);
}

public Hashtable(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load: "+loadFactor);

    if (initialCapacity==0)
        initialCapacity = 1;
    this.loadFactor = loadFactor;
    // 始化table，获得大小为initialCapacity的table数组
    table = new Entry<?,?>[initialCapacity];
    // 计算阀值
    threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}
```

* **扩容**

```java
// hastable扩容分析：
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        // 判断value是否为null
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    // key若为null也会抛出异常
    int hash = key.hashCode();
    // 计算key在hash表中的位置
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    // entry为null，表示还没有在hash表中添加过元素，需要去添加
    // entry不为null，表示发生了hash冲突，则遍历链表找到插入的位置
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            // key已经存在了，则替换一下value，然后返回oldvalue
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }
	
    // 添加元素，见下面
    addEntry(hash, key, value, index);
    return null;
}

private void addEntry(int hash, K key, V value, int index) {
    modCount++;

    Entry<?,?> tab[] = table;
    // 对容量进行检验，若大于阈值，则需要进行扩容操作，见下面
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();

        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }
	
    // Creates the new entry.
    @SuppressWarnings("unchecked")
    // 添加元素
    Entry<K,V> e = (Entry<K,V>) tab[index];
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}

// rehash，扩容操作
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;

    // overflow-conscious code
    // 这里：新容量=旧容量 * 2 + 1
    int newCapacity = (oldCapacity << 1) + 1;
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            // Keep running with MAX_ARRAY_SIZE buckets
            return;
        newCapacity = MAX_ARRAY_SIZE;
    }
    // 新建一个size = newCapacity 的HashTable
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

    modCount++;
    // 重新计算阀值
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    table = newMap;

    for (int i = oldCapacity ; i-- > 0 ;) {
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;
            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```

## 参考

https://blog.csdn.net/u010983881/article/details/49762595

https://mp.weixin.qq.com/s/q1r9Pno6ANUzZ9wMzA-JSg