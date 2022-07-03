# Java：容器类线程不安全

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

## 1. Collection 线程不安全的举例

### 前言

当我们执行下面语句的时候，底层进行了什么操作

```java
new ArrayList<Integer>();
```

底层创建了一个空的数组，伴随着初始值为 10

当执行 add 方法后，如果超过了 10，那么会进行扩容，扩容的大小为原值的一半，也就是 5 个，使用下列方法扩容

```java
Arrays.copyOf(elementData, netCapacity)
```

### 单线程环境下

单线程环境的 ArrayList 是不会有问题的

```java
public class ArrayListNotSafeDemo {
    public static void main(String[] args) {

        List<String> list = new ArrayList<>();
        list.add("a");
        list.add("b");
        list.add("c");

        for(String element : list) {
            System.out.println(element);
        }
    }
}
```

### 多线程环境

为什么 ArrayList 是线程不安全的？因为在进行写操作的时候，方法上为了保证并发性，是没有添加synchronized 修饰，所以并发写的时候，就会出现问题

```java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

当我们同时启动30个线程去操作List的时候

```java
/**
 * 集合类线程不安全举例
 */
public class ArrayListNotSafeDemo {
    public static void main(String[] args) {

        List<String> list = new ArrayList<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }
}
```

这个时候出现了错误，也就是 **java.util.ConcurrentModificationException**

这个异常是 **并发修改的异常**

### 解决方案

#### 方案一：Vector

第一种方法，就是不用 ArrayList 这种不安全的 List 实现类，而采用 Vector，线程安全的

关于 Vector 如何实现线程安全的，而是在方法上加了锁，即 synchronized

```java
/**
 * Adds the specified component to the end of this vector,
 * increasing its size by one. The capacity of this vector is
 * increased if its size becomes greater than its capacity.
 *
 * <p>This method is identical in functionality to the
 * {@link #add(Object) add(E)}
 * method (which is part of the {@link List} interface).
 *
 * @param   obj   the component to be added
 */
public synchronized void addElement(E obj) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = obj;
}
```

这样就每次只能够一个线程进行操作，所以不会出现线程不安全的问题，但是因为加锁了，导致并发性基于下降

#### 方案二：Collections.synchronizedXXX

```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
```

采用 Collections 集合工具类，在 ArrayList 外面包装一层同步机制

#### 方案三：采用 JUC 里面的方法

CopyOnWriteArrayList：写时复制，主要是一种**读写分离的思想**

写时复制，CopyOnWrite 容器即写时复制的容器，往一个容器中添加元素的时候，不直接往当前容器 `Object[]` 添加，而是先将 `Object[]` 进行 copy，复制出一个新的容器 `object[] newElements`，然后新的容器 `Object[] newElements` 里添加原始，添加元素完后，在将原容器的引用指向新的容器 `setArray(newElements);` 这样做的好处是可以对 copyOnWrite 容器进行并发的读，而不需要加锁，因为当前容器不需要添加任何元素。

所以 CopyOnWrite 容器也是一种读写分离的思想，读和写不同的容器，就是写的时候，把 ArrayList 扩容一个出来，然后把值填写上去，再通知其他的线程，ArrayList 的引用指向扩容后的容器。

查看底层 `add()` 方法源码

```java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 获取当前数组
        Object[] elements = getArray();
        int len = elements.length;
        // 新创建一个数组
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 把元素添加上去
        newElements[len] = e;
        // 设置新的数组为后续使用的数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

首先需要加锁

```java
final ReentrantLock lock = this.lock;
lock.lock();
```

然后在末尾扩容一个单位

```java
Object[] elements = getArray();
int len = elements.length;
Object[] newElements = Arrays.copyOf(elements, len + 1);
```

然后在把扩容后的空间，填写上需要add的内容

```java
newElements[len] = e;
```

最后把内容set到Array中

```java
setArray(newElements);
```

## 2. HashSet 线程不安全

### CopyOnWriteArraySet

> 加了马甲的 CopyOnWriteArrayList 

底层还是使用 CopyOnWriteArrayList 进行实例化

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    private final CopyOnWriteArrayList<E> al;

    /**
     * Creates an empty set.
     */
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
}
```

### HashSet 底层结构

同理HashSet的底层结构就是HashMap

```java
/**
 * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
 * default initial capacity (16) and load factor (0.75).
 */
public HashSet() {
    map = new HashMap<>();
}
```

问：但是为什么我调用 HashSet.add()的方法，只需要传递一个元素，而HashMap是需要传递key-value键值对？

首先我们查看hashSet的add方法

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

// 其中：
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
```

我们能发现但我们调用 add 的时候，存储一个值进入 map 中，只是作为 key 进行存储，而 value 存储的是一个 Object 类型的常量，也就是说 HashSet 只关心 key，而不关心 value

## 3. HashMap 线程不安全

同理 HashMap 在多线程环境下，也是不安全的

```java
public static void main(String[] args) {

    Map<String, String> map = new HashMap<>();

    for(int i = 0; i < 30; i++){
        new Thread(() -> {
            map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0, 8));
            System.out.println(map);
        }, String.valueOf(i)).start();
    }
}
```

### 解决方法

1、使用 `Collections.synchronizedMap(new HashMap<>());`

2、使用 `ConcurrentHashMap`

```java
Map<String, String> map = new ConcurrentHashMap<>();
```

