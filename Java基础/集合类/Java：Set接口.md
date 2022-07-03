# Java：Set接口

> 对 Java 中的 **Set接口 与 其实现类**，做一个微不足道的小小小小记

## 概述

```java
public interface Set<E> extends Collection<E> {
	// ...
}
```

Set 接口继承了 Collection 接口，Set 集合中不能包含重复的元素，每个元素必须是唯一的，你只要将元素加入 Set 中，重复的元素会自动移除。

> 对于 Set 的个人理解：
>
> 1. 使用上：就是不能带有重复元素的 List；
> 2. 本质上：只用了 key 的 Map，一个穿了马甲的 Map

## HashSet

### 概述

> 披着马甲的 HashMap：[Java：HashMap类小记](https://www.cnblogs.com/zhuchengchao/p/14298141.html)
>
> ```java
> public HashSet() {
>        // 在HashSet的构造函数中，new了一个HashMap
>        map = new HashMap<>();
> }
> ```

1. HashSet 中不能有相同的元素，但**可以有 null 元素**，即元素具有唯一性；

   其底层采用的还是 HashMap，对于 put 元素，其采用的方式为：

   ```java
   // HashSet的底层为一个HashMap结构
   private transient HashMap<E, Object> map;
   // 作为一个占位符
   // private static final Object PRESENT = new Object(); 
   
   public HashSet() {
       map = new HashMap<>();
   }
   public boolean add(E e) {
       return map.put(e, PRESENT)==null;
   }
   ```

   > 从上述 add 方法可知：**HashSet 保证元素不重复的方式**
   >
   > 元素值作为的是 map 的 key，map 的 value 则是 PRESENT 变量，这个变量只作为放入 map 时的一个占位符而存在，所以没什么实际用处。
   >
   > 其实，这时候答案已经出来了：**HashMap 的 key 是不能重复的，而这里 HashSet 的元素又是作为了 map 的 key，当然也不能重复了。**
   >
   > LinkedHashSet、TreeSet 都一样，都采用了 PRESENT 这个占位符。

2. 存入 HashSet 的元素是无序的；

3. 添加、删除元素的操作时间复杂度都为 O(1)；

4. 非线程安全。

### 实现原理

1. HashSet 的实现是依赖于 HashMap 的，HashSet 的值都是存储在 HashMap 中的；

   ```java
   public class HashSet<E>
       extends AbstractSet<E>
       implements Set<E>, Cloneable, java.io.Serializable{}
   ```

2. 在 HashSet 的构造函数中会初始化一个 HashMap 对象，HashSet 的值是作为 HashMap 的 key 存储在 HashMap 中的，当存储的值已经存在时返回 false；

   ```java
   private transient HashMap<E,Object> map;
   public HashSet() {
       map = new HashMap<>();
   }
   public boolean add(E e) {
       // add时，当值以及存在，则map返回的是旧值，不为null，则返回false，表示插入的数据是重复的
       return map.put(e, PRESENT)==null;
       // private static final Object PRESENT = new Object();
   }
   ```

3. HashSet 的其他操作都是基于 HashMap 的

   ```java
   public int size() {
       return map.size();
   }
   public boolean isEmpty() {
       return map.isEmpty();
   }
   public boolean add(E e) {
       return map.put(e, PRESENT)==null;
   }
   public boolean remove(Object o) {
       return map.remove(o)==PRESENT;
   }
   // ....
   ```

## LinkedHashSet

### 概述

> 披着马甲的 LinkedHashMap：[Java：LinkedHashMap类小记](https://www.cnblogs.com/zhuchengchao/p/14447418.html)
>
> ```java
> public class LinkedHashSet<E>
>        extends HashSet<E>
>        implements Set<E>, Cloneable, java.io.Serializable {
> 
>        // 1.LinkedHashSet的构造函数，调用了其父类HashSet的构造函数
>        public LinkedHashSet(int initialCapacity, float loadFactor) {
>            // super：用了HashSet中的构造函数，且第三个参数为true
>            // 其他的构造函数也相同，第三个元素都为true
>            super(initialCapacity, loadFactor, true);
>        }
> }
> 
> // 2.进入HashSet查看对应的构造函数
> HashSet(int initialCapacity, float loadFactor, boolean dummy) {
>        // 这里new了一个LinkedHashMap
>        map = new LinkedHashMap<>(initialCapacity, loadFactor);
> }
> 
> // 3.进入LinkedHashMap中查看对应的构造函数
> public LinkedHashMap(int initialCapacity, float loadFactor) {
>        // 这里调用了LinkedHashMap的父类HashMap的构造函数
>        super(initialCapacity, loadFactor);
>        accessOrder = false;
> }
> 
> // 4.查看HashMap的构造函数
> public HashMap(int initialCapacity, float loadFactor) {
>        if (initialCapacity < 0)
>            throw new IllegalArgumentException("Illegal initial capacity: " +
>                                               initialCapacity);
>        if (initialCapacity > MAXIMUM_CAPACITY)
>            initialCapacity = MAXIMUM_CAPACITY;
>        if (loadFactor <= 0 || Float.isNaN(loadFactor))
>            throw new IllegalArgumentException("Illegal load factor: " +
>                                               loadFactor);
>        this.loadFactor = loadFactor;
>        this.threshold = tableSizeFor(initialCapacity);
> }
> ```

1. LinkedHashSet 中不能有相同元素，**可以有一个 null 元素**，元素严格按照放入的顺序排列；
2. 添加、删除元素的操作时间复杂度都为O(1)；
3. 非线程安全。

### 实现原理

1. 对于 LinkedHashSet 而言，它继承于 HashSet，又基于 LinkedHashMap 来实现的；

   > 具体见上面构造一个 LinkedHashSet 调用的构造函数过程
   
2. LinkedHashSet 底层使用 LinkedHashMap 来保存所有元素，其所有的方法操作上又基本都是使用了 HashSet 的实现；

   > LinkedHashSet 内部除了几个构造函数，基本就没什么方法了，都是用了父类 HashSet 的方法

## TreeSet

### 概述

> 披着马甲的 TreeSet：[Java：TreeMap类小记](https://www.cnblogs.com/zhuchengchao/p/14448137.html)
>
> ```java
> // TreeSet成员属性
> private transient NavigableMap<E,Object> m;
> // 占位符
> private static final Object PRESENT = new Object();
> // TreeSet的构造函数
> public TreeSet() {
>     // 在构造函数中new了一个TreeMap
>     this(new TreeMap<E,Object>());
> }
> ```

1. TreeSet 是中不能有相同元素，**不可以有null元素**（因为需要根据key进行比较），根据元素的自然顺序进行排序；
2. 添加、删除操作时间复杂度都是 O(log(n))；
3. 非线程安全

### 实现原理

1. TreeSet 底层实际使用的存储容器就是 TreeMap；

   ```java
   public class TreeSet<E> extends AbstractSet<E>
       implements NavigableSet<E>, Cloneable, java.io.Serializable
   {
       // 成员属性
   	private transient NavigableMap<E,Object> m;
   
   	public TreeSet() {
   		// 底层实际使用的存储容器就是 TreeMap；
           this(new TreeMap<E,Object>());  // 调用 TreeMap
       }
       
       // 把TreeMap赋值给了NavigableMap<E,Object>
       TreeSet(NavigableMap<E,Object> m) {
           this.m = m;
       }
       
       public boolean add(E e) {
       	// 本质上还是调用了 TreeMap 的方法
           return m.put(e, PRESENT)==null;
       }
   }
   ```

2. TreeSet 里绝大部分方法都是直接调用 TreeMap 的方法来实现的；

## 总结

首先，Set 集合不包含重复元素，都具有唯一性；

当无排序要求可以选用 HashSet；

若想取出元素的顺序和放入元素的顺序相同，那么可以选用 LinkedHashSet；

若想插入、删除立即排序或者按照一定规则排序可以选用 TreeSet。

## 参考

https://blog.csdn.net/StemQ/article/details/66477615

https://mp.weixin.qq.com/s/q1r9Pno6ANUzZ9wMzA-JSg

https://blog.csdn.net/qq_41026809/article/details/90449073

https://www.cnblogs.com/baojun/p/11087004.html

https://blog.csdn.net/jtcode_is_my_partner/article/details/81408392