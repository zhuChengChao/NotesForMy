# Java：ArrayList类

> 对 Java 中的 **ArrayList类**，做一个微不足道的小小小小记

## 概述

`java.util.ArrayList` 是**大小可变的数组**的实现，存储在内的数据称为元素。此类提供一些方法来操作内部存储的元素。 ArrayList 中可不断添加元素，其大小也自动增长。

* `ArrayList` 是一种变长的集合类，基于定长数组实现。
* `ArrayList` 允许空值和重复元素，当往 ArrayList 中添加的元素数量大于其底层数组容量时，其会通过**扩容**机制重新生成一个更大的数组。
* 由于 `ArrayList` 底层基于数组实现，所以其可以保证在 `O(1)` 复杂度下完成随机查找操作。
* `ArrayList` 是非线程安全类，并发环境下，多个线程同时操作 ArrayList，会引发不可预知的异常或错误。

## 成员属性

```java
// 序列化号
private static final long serialVersionUID = 8683452581122892189L;

// 默认的容量
private static final int DEFAULT_CAPACITY = 10;

// 默认的空的数组，这个主要是在构造方法初始化一个空数组的时候使用
private static final Object[] EMPTY_ELEMENTDATA = {};

// 使用默认size大小的空数组实例，和EMPTY_ELEMENTDATA区分开来，
// 这样可以知道当第一个元素添加的时候进行扩容至多少
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

// ArrayList底层存储数据就是通过数组的形式，ArrayList长度就是数组的长度。
// 一个空的实例elementData为上面的DEFAULTCAPACITY_EMPTY_ELEMENTDATA，当添加第一个元素的时候会进行扩容，扩容大小就是上面的默认容量DEFAULT_CAPACITY
transient Object[] elementData;

// ArrayList的大小
private int size;
```

## 构造函数

**带初始容量的构造方法**

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        // elementData初始化为initialCapacity大小的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        // 如果传递的长度为0，就是直接使用自己已经定义的成员变量(一个空数组)
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        // 抛出异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

**无参构造**

```javascript
public ArrayList() {
    // 构造方法中将elementData初始化为空数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    // 这样可以知道当第一个元素添加的时候进行扩容至多少
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

**参数为Collection类型的构造器**

```java
public ArrayList(Collection<? extends E> c) {
    // 将一个参数为Collection的集合转变为ArrayList
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            // c.toArray()可能不会正确地返回一个 Object[]数组，那么使用Arrays.copyOf()方法
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        // 如果集合转换为数组之后数组长度为0，就直接使用自己的空成员变量初始化elementData
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

## ArrayList 的扩容机制

### 末尾添加

1. 当使用 add 方法的时候首先调用 ensureCapacityInternal 方法，传入 size+1 进去，检查是否需要扩充 elementData 数组的大小；

   ```java
   // 1.ArrayList中的add方法
   public boolean add(E e) {
       // 因为要添加元素，所以添加之后可能导致容量不够，所以需要在添加之前进行判断（扩容）
       ensureCapacityInternal(size + 1);  // Increments modCount!!
       // 最后扩容成功/不需要扩容  才添加元素，并令size+1
       elementData[size++] = e;
       return true;
   }
   
   // 2.ArrayList 中 add 方法的 ensureCapacityInternal 方法
   private void ensureCapacityInternal(int minCapacity) {
       ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
   }
   
   // 3.calculateCapacity方法：
   // 判断传入的数组是否为空，若是，则返回DEFAULT_CAPACITY与minCapacity中的最大值
   private static int calculateCapacity(Object[] elementData, int minCapacity) {
       // 其中，DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
       // 满足下面条件表示的是第一次添加元素需要的扩容操作
       if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
           // 其中private static final int DEFAULT_CAPACITY = 10;
           return Math.max(DEFAULT_CAPACITY, minCapacity);
       }
       return minCapacity;
   }
   
   // 4.ensureExplicitCapacity方法：
   // 判断添加元素后的数组，是否可能发生溢出，若可能发生溢出，则进行grow扩容
   private void ensureExplicitCapacity(int minCapacity) {
       modCount++;  // 涉及 fail-fast
   
       // overflow-conscious code 已经溢出了 需要扩容了
       if (minCapacity - elementData.length > 0)
           grow(minCapacity);
   }
   ```

2. 当需要扩容，即已经发生了溢出，则调用grow函数进行扩容操作

   ```java
   private void grow(int minCapacity) {
       // overflow-conscious code
       int oldCapacity = elementData.length;
       // 扩容大小为原数组长度的1.5倍；
       int newCapacity = oldCapacity + (oldCapacity >> 1);
       // 检查新容量的大小是否小于最小需要容量，如果小于那就将最小容量最为数组的新容量
       // 这是考虑到第一次进入时，oldCapacity为0的情况
       if (newCapacity - minCapacity < 0)
           newCapacity = minCapacity;
       // 当新容量大于MAX_ARRAY_SIZE，使用hugeCapacity比较二者
       if (newCapacity - MAX_ARRAY_SIZE > 0)
           newCapacity = hugeCapacity(minCapacity);
       // minCapacity is usually close to size, so this is a win:
       elementData = Arrays.copyOf(elementData, newCapacity);
   }
   ```

   其中：`newCapacity = oldCapacity + (oldCapacity >> 1);`扩容大小为原数组长度的1.5倍；

   `newCapacity - MAX_ARRAY_SIZE`当新容量大于 MAX_ARRAY_SIZE，使用hugeCapacity比较二者

   ```java
   private static int hugeCapacity(int minCapacity) {
       if (minCapacity < 0) // overflow
           throw new OutOfMemoryError();
       // 对minCapacity和MAX_ARRAY_SIZE进行比较
       // 若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
       // 若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
       // 其中：MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
       return (minCapacity > MAX_ARRAY_SIZE) ?
           Integer.MAX_VALUE :
       MAX_ARRAY_SIZE;
   }
   ```

3. ArrayList 中 copy 数组的核心就是 `System.arraycopy` 方法，将 original 数组的所有数据复制到 copy 数组中，这是一个本地方法。

   ```java
   // 在grow函数中最终调用Arrays.copyOf方法
   elementData = Arrays.copyOf(elementData, newCapacity);
   
   // 进一步：Arrays.copyOf 调用流程
   // 1.Arrays类中的copyOf方法：
   @SuppressWarnings("unchecked")
   public static <T> T[] copyOf(T[] original, int newLength) {
       return (T[]) copyOf(original, newLength, original.getClass());
   }
   
   // 2.Arrays类中的copyOf方法：
   public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
       @SuppressWarnings("unchecked")
       T[] copy = ((Object)newType == (Object)Object[].class)
           ? (T[]) new Object[newLength]
           : (T[]) Array.newInstance(newType.getComponentType(), newLength);
       System.arraycopy(original, 0, copy, 0,
                        Math.min(original.length, newLength));
       return copy;
   }
   
   // 3. System类中的arraycopy方法：是一个native方法
   public static native void arraycopy(Object src,  int  srcPos,
                                       Object dest, int destPos,
                                       int length);
   ```

**总的来说就是分两步：**

1. 扩容：把原来的数组复制到另一个内存空间更大的数组中
2. 添加元素：把新元素添加到扩容以后的数组中

### 添加到指定位置

上述通过对`public boolean add(E e)`函数的分析，大致了解一下 ArrayList 的扩容机制，`add(E e)`函数为在最后添加元素，当然还有方法`public void add(int index, E element)`在指定的位置添加元素：

```java
public void add(int index, E element) {
    // 1.校验传递的index参数是不是合法
    rangeCheckForAdd(index);
	
    // 2.还是检测是否需要扩容，上面已经分析过了
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 3.将 index 及其之后的所有元素都向后移一位
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 4.将新元素插入至 index 处
    elementData[index] = element;
    size++;
}

// 上述的rangeCheckForAdd方法：
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

## 移除元素

ArrayList支持两种删除元素的方式：

* `public E remove(int index)`：按照下标移除
* `public boolean remove(Object o)`：按照元素移除

### 按照下标删除

```java
public E remove(int index) {
    // 校验下标是否合法
    rangeCheck(index);
	// fail-fast机制
    modCount++;
    // 直接在数组中查找这个值
    E oldValue = elementData(index);
	
    // 这里计算所需要移动的数目
    int numMoved = size - index - 1;
    // 如果这个值大于0 说明后续有元素需要左移(size=index+1)
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 移动之后，原数组中size位置null
    elementData[--size] = null; // clear to let GC do its work
	// 返回旧值
    return oldValue;
}
```

### 按元素删除

按照元素删除，会删除和参数匹配的第一个元素

```java
public boolean remove(Object o) {
    // 如果元素是null 遍历数组移除第一个null
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                // 遍历找到第一个null元素的下标 调用下标移除元素的方法
                fastRemove(index);
                // 直接return了，说明的就是删除第一个找到的元素
                return true;
            }
    } else {
        // 找到元素对应的下标 调用下标移除元素的方法
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// 上述的按照索引删除元素的方法：
private void fastRemove(int index) {
    modCount++;
    // 这里计算所需要移动的数目
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

## Array & ArrayList

**区别：**

1. Array 可以容纳基本类型和对象，而 ArrayList 只能容纳对象，但是 Array 数组在存放的时候一定是同种类型的元素，而 ArrayList 就不一定了，因为 ArrayList 可以存储 Object 类型的对象；
2. Array 大小是固定的，ArrayList 的大小是动态变化的（动态扩容）。

**什么时候更适合使用 Array：**

1. 如果列表的大小已经指定，大部分情况下是存储和遍历它们；
2. 对于遍历基本数据类型，尽管 Collections 使用自动装箱来减轻编码任务，在指定大小的基本类型的列表上工作也会变得很慢；
3. 如果你要使用多维数组，使用 `[][]` 比`<List>` 更容易。

## RandomAccess 接口

可以看到 ArrayList 实现了 RandomAccess 接口

```java
public class ArrayList<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
    // ...
}
```

而 LinkedList 未实现该接口

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable{
    // ...
}
```

**说明原因：**

1. RandomAccess 接口只是一个标志接口(Marker interface)，只要 List 集合实现这个接口，就能**支持快速随机访问**。

2. 通过查看 Collections 类中的 `copy()` 方法：

   ```java
   public static <T> void copy(List<? super T> dest, List<? extends T> src) {
       int srcSize = src.size();
       if (srcSize > dest.size())
           throw new IndexOutOfBoundsException("Source does not fit in dest");
   
       if (srcSize < COPY_THRESHOLD ||
           (src instanceof RandomAccess && dest instanceof RandomAccess)) {
           // src与dest实现了RandomAccess接口，则用for循环遍历
           for (int i=0; i<srcSize; i++)
               dest.set(i, src.get(i));
       } else {
           // src 或 dest 有一没实现RandomAccess接口，则用iterator循环遍历
           ListIterator<? super T> di=dest.listIterator();
           ListIterator<? extends T> si=src.listIterator();
           for (int i=0; i<srcSize; i++) {
               di.next();
               di.set(si.next());
           }
       }
   }
   ```

   实现 RandomAccess 接口的 List 集合采用一般的 for 循环遍历，而未实现这接口则采用迭代器，即 ArrayList 一般采用 for 循环遍历，而 LinkedList 一般采用迭代器遍历；

3. **ArrayList 用 for 循环遍历比 iterator 迭代器遍历快，LinkedList 用 iterator 迭代器遍历比 for 循环遍历快。**所以说，当我们在做项目时，应该考虑到 List 集合的不同子类采用不同的遍历方式，能够提高性能。

## ArrayList & LinkedList

虽然还没看过 LinkedList 的源码，这里就先贴一个 ArrayList & LinkedList 的区别

**ArrayList：**底层是基于动态数组实现的，**查找快，增删较慢**；

**LinkedList：**底层是基于链表实现的。确切的说是**循环双向链表**（JDK1.6 之前是双向循环链表、JDK1.7 之后取消了循环），**查找慢、增删快**。LinkedList 链表由一系列表项连接而成，**一个表项包含 3 个部分：元素内容、前驱表和后驱表。**链表内部有一个 header 表项，既是链表的开始也是链表的结尾。header 的后继表项是链表中的第一个元素，header 的前驱表项是链表中的最后一个元素。

> **补充：**
>
> **ArrayList 的增删未必就是比 LinkedList 要慢：**
>
> 1. 如果增删都是在末尾来操作（每次调用的都是 `remove()` 和 `add()`），此时 ArrayList 就不需要移动和复制数组来进行操作了。如果数据量有百万级的时，速度是会比 LinkedList 要快的。
>
> 2. 如果删除操作的位置是在中间。由于 LinkedList 的消耗主要是在遍历上（ LinkedList 会比较查询的index与链表长度，选择从头还是从尾开始查找），ArrayList 的消耗主要是在移动和复制上（**底层调用的是 `arrayCopy()` 方法，是 native 方法，native的方法还是很快的**）。LinkedList 的遍历速度是要慢于 ArrayList 的复制移动速度的，如果数据量有百万级的时，还是 ArrayList 要快。
>
>    ```java
>    // ArrayList
>    public void add(int index, E element) {
>        rangeCheckForAdd(index);
>       
>        ensureCapacityInternal(size + 1);  // Increments modCount!!
>        // 这个是一个native的方法
>        System.arraycopy(elementData, index, elementData, index + 1,
>                         size - index);
>        elementData[index] = element;
>        size++;
>    }
>       
>    // 比较需要插入的 index 与链表长度
>    // 小于链表长度一般则从而开始遍历，反之从尾开始遍历
>    Node<E> node(int index) {
>        // assert isElementIndex(index);
>       
>        if (index < (size >> 1)) {   // 小于链表长度的一半，则从头开始遍历
>            Node<E> x = first;
>            for (int i = 0; i < index; i++)
>                x = x.next;
>            return x;
>        } else {					// 大于链表长度的一半，则从末尾开始遍历
>            Node<E> x = last;
>            for (int i = size - 1; i > index; i--)
>                x = x.prev;
>            return x;
>        }
>    }
>    ```
>
> > 参考：https://www.cnblogs.com/syp172654682/p/9817277.html

## 参考：

☆☆☆：https://juejin.cn/post/6844903904346374158#heading-2

https://www.cnblogs.com/SunArmy/p/9844022.html

https://mp.weixin.qq.com/s/q1r9Pno6ANUzZ9wMzA-JSg

https://blog.csdn.net/weixin_39148512/article/details/79234817