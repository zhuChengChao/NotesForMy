# Java：LinkedList类

> 对 Java 中的 **LinkedList类**，做一个微不足道的小小小小记

## 概述

`java.util.LinkedList` 集合数据存储的结构是循环双向链表结构。方便元素添加、删除的集合。

**循环双向链表：**

1. **双向**：链表中任意一个存储单元都可以通过向前或者向后寻址的方式获取到其前一个存储单元和其后一个存储单元

2. **循环**：链表的尾节点的后一个节点是链表的头结点，链表的头结点的前一个节点是链表的尾节点

可以画出如下示意图：

![循环双向链表](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251615489.JPG)

> JDK1.6 之前是双向循环链表、**JDK1.7 之后取消了循环**
>
> 就是这样的结构，使得链表可以作为**队列/双端队列**使用，在刷题的时候无敌 :dog:

## 成员属性

LinkedList 继承了 AbstractSequentialList 类，实现了 List、Deque(双向队列) 接口

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;
	// 双向链表的头结点
    transient Node<E> first;
    // 双向链表的尾节点
    transient Node<E> last;
	
    // 静态内部类 Node
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
}
```

## 常用方法

### 构造方法

```java
// 默认/无参构造
public LinkedList() {
}

// 根据其他的集合类构造
public LinkedList(Collection<? extends E> c) {
    this();     // 调用了上面的无参构造
    addAll(c);  
}
```

### 添加元素

**`add(E e)`在链表末尾插入元素：**

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    // 获取末尾节点
    final Node<E> l = last;
    // 创建新节点
    final Node<E> newNode = new Node<>(l, e, null);
    // 更新末尾节点为新节点
    last = newNode;
    // 当之前的末尾节点为null，说明在add时是个空链表
    if (l == null)
        // 链表头和尾现在都是这个newnode
        first = newNode;
    else
        // 链表不为空，之前的last.next直线新的last
        l.next = newNode;
    size++;
    modCount++;
}
```

**`add(int index, E element)`在指定位置插入元素**

```java
public void add(int index, E element) {
    // 1.判断index是否合法
    checkPositionIndex(index);

    if (index == size)
        // 2.当index就是size，则在末尾插入,就是上面的方法
        linkLast(element);
    else
        // 3.在链表的指定位置插入元素
        // 3.1 node
        // 3.2 linkBefore
        linkBefore(element, node(index));
}

// 1.1 判断index是否合法
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
// 1.2上述的isPositionIndex方法：
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}

// 3.1 根据index找到对应的node位置
// 比较需要插入的 index 与链表长度
// 小于链表长度一般则从而开始遍历，反之从尾开始遍历
Node<E> node(int index) {
    // assert isElementIndex(index);
	
    // index小于链表的一半，则从头开始遍历链表
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

// 3.2 在链表的指定位置插入元素
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

### 移除元素

**`remove(Object o)`移除指定元素：**

```java
public boolean remove(Object o) {
    if (o == null) {
        // 当o为null时，移除第一个为null的元素
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        // 当o不为null时，移除相应的元素
        for (Node<E> x = first; x != null; x = x.next) {
            // 这里的通过equals的方法进行判断
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

// 上述函数中的unlink函数
// 主要的逻辑就是：删除某节点，并将该节点的上一个节点（如果有）和下一个节点（如果有）关联起来
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;  // 用于返回待删除节点的值
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

**`remove(int index)`移除指定索引的元素：**

```java
public E remove(int index) {
    // 判断index是否合法
    checkElementIndex(index);
    // 修改节点的连接
    return unlink(node(index));
}
```

### 其他添加/删除方法：

**和队列相关的一些方法：**

`public boolean offer(E e)`：在 list 末尾添加上元素；

`public boolean offerFirst(E e)`：在 list 头添加元素；

`public boolean offerLast(E e)`:在 list 末尾添加上元素，本质上和 offer 方法是一样的；

`public E poll()`：出 list 头的元素，并返回；

`public E pollFirst()`：出 list 头的元素，并返回；

`public E pollLast()`：出 list 末尾的元素，并返回；

`public E peek()`：返回 list 头部的元素；

`public E peekFirst()`：返回 list 头部的元素；

`public E peekLast()`：返回 list 尾部的元素；

**和栈相关的一些方法：**

`public void push(E e)`：栈的push操作

`public E pop()`：栈的pop操作

## ArrayList & LinkedList

> 这部分内容在[Java：ArrayList类小记](https://www.cnblogs.com/zhuchengchao/p/14297634.html)中已经贴了，这里再贴一遍

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

