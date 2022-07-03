# Java：Iterator&fail-fast

> 对 Java 中的 **Iterator接口 和 fail-fast**，做一个微不足道的小小小小记

## Iterator

### Iterator 接口

**Iterator：迭代器**

迭代器是一种设计模式，它是一个对象，它可以遍历并选择序列中的对象，而开发人员不需要了解该序列的底层结构。迭代器通常被称为“轻量级”对象，因为创建它的代价小。

Java 中的 Iterator 功能比较简单，并且只能单向移动：　　

1. 使用方法 `public Iterator iterator()` 要求容器返回一个 Iterator。第一次调用 Iterator 的 `next()` 方法时，它返回序列的第一个元素。注意：`iterator()` 方法是 `java.lang.Iterable` 接口，被 Collection 继承。　　
2. 使用 `public E next()` 获得序列中的下一个元素。　
3. 使用 `public boolean hasNext()` 检查序列中是否还有元素。　　
4. 使用 `default void remove()` 将迭代器新返回的元素删除。

**进一步：**

**迭代**：即 Collection 集合元素的通用获取方式。在取元素之前先要判断集合中有没有元素，如果有，就把这个元素取出来，继续在判断，如果还有就再取出出来。一直把集合中的所有元素全部取出。这种取出方式专业术语称为迭代。

```java
public class IteratorDemo {
    public static void main(String[] args) {
        // 使用多态方式 创建对象
        Collection<String> coll = new ArrayList<String>();
        // 添加元素到集合
        coll.add("串串星人");
        coll.add("吐槽星人");
        coll.add("汪星人");
        // 遍历
        // 使用迭代器 遍历 每个集合对象都有自己的迭代器
        Iterator<String> it = coll.iterator();
        // 泛型指的是 迭代出 元素的数据类型
        while(it.hasNext()){ //判断是否有迭代元素
            String s = it.next();//获取迭代出的元素
            System.out.println(s);
        }
    }
}
```

> 注：在进行集合元素取出时，如果集合中已经没有元素了，还继续使用迭代器的next方法，将会发生`java.util.NoSuchElementException`没有集合元素的错误。

### ListIterator

**ListIterator 与 Iterator 的相同点：**

1. 都是迭代器，当需要对集合中元素进行遍历不需要干涉其遍历过程时，这两种迭代器都可以使用；

2. ListIterator 实现了 Iterator 接口；

   ```java
   public interface ListIterator<E> extends Iterator<E>
   ```

**不同点：**

1. 使用范围不同：Iterator 可用来遍历 Set、List、Map集合，但是 **ListIterator 只能用来遍历 List**。
2. Iterator 对集合只能是前向遍历（`hasNext()`, `Next()`），ListIterator 既可以前向也可以后向(`hasNext()`, `Next()`, `hasPrevious()`, `previous()`)；
3. ListIterator 可以定位当前索引的位置，`nextIndex()`和`previousIndex()`可以实现。Iterator没有此功能。
4. ListIterator 有 add 方法，可以向 List 中添加对象，而 Iterator 不能；
5. 都可实现删除操作，但是 ListIterator 可以实现对象的修改，`set()`方法可以实现。Iterator仅能遍历，不能修改

**总之**：ListIterator 实现了 Iterator 接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引等等。

### Enumeration

与 Enumeration 相比，Iterator 更加安全，因为当一个集合正在被遍历的时候，它会阻止其它线程去修改集合。否则会抛出 `ConcurrentModificationException` 异常。这其实就是 **fail-fast 机制**。具体区别有三点：

1. Iterator 的方法名比 Enumeration 更科学；
2. **Iterator 有 fail-fast 机制，比 Enumeration 更安全**；
3. Iterator 能够删除元素，Enumeration 并不能删除元素。

函数接口如下：

```java
package java.util;

public interface Enumeration<E> {
    boolean hasMoreElements();
    E nextElement();
}
public interface Iterator<E> {
    boolean hasNext();
    E next();
    void remove();
}
```

## fail-fast

在上述 Iterator 中，介绍到 Enumeration 时，提到了 fail-fast 机制，在此做一点介绍

### fail-fast 概述

fail-fast 的字面意思是“快速失败”。当我们在遍历集合元素的时候，经常会使用迭代器，但在迭代器遍历元素的过程中，如果集合的结构被改变的话，就会抛出异常，防止继续遍历。这就是所谓的快速失败机制。

fail-fast 迭代器抛出 `ConcurrentModificationException`，而 fail-safe 迭代器则不会。

> 结构上的改变：集合上的插入和删除就是结构上的改变；
>
> 对集合中某个元素进行修改的话，并不是结构上的改变；

抛出 ConcurrentModificationException 异常实例：

```java
@Test
public void testFailFast(){
    List<Integer> list = new ArrayList<>();
    for(int i = 0; i < 20; i++){
        list.add(i);
    }
    Iterator<Integer> it = list.iterator();
    int temp = 0;
    while(it.hasNext()){
        if(temp == 3){
            temp++;
            list.remove(3);  // 在迭代的过程中，改变了集合的结构，导致fail-fast
        }else{
            temp++;
            System.out.println(it.next());
        }
    }
}
```

### fail-fast 工作原理

```java
public E next() {
	checkForComodification();
    ...
}

final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

当迭代器在执行 `next()` 方法时，会调用 `checkForComodification()`，当`modCount != expectedModCount` 时则抛出异常，而当集合的结构发生变化时，modCount 就会发生改变，如：

```java
// ArrayList 在添加元素时，modCount就会被改变
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;  // 这里修改了！！！

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

**fail-fast 的一些处理方法：**

如果我们不希望在迭代器遍历的时候因为并发等原因，导致集合的结构被改变，进而可能抛出异常的话，我们可以在涉及到会影响到 modCount 值改变的地方，加上同步锁(synchronized),或者直接使用 `Collections.synchronizedList` 来解决。

### fail-safe

java.util 包中的所有集合类都被设计为 fail-fast 的，而 java.util.concurrent 中的集合类都为 fail-safe 的。

对于采用 fail-safe 机制来说，当检测到正在遍历的集合的结构被改变时，不会抛出异常；

这是因为，**当集合的结构被改变的时候，fail-safe 机制会在复制原集合的一份数据出来**，然后在复制的那份数据遍历。

因此，虽然fail-safe不会抛出异常，但存在以下缺点：

1. 复制时需要额外的空间和时间上的开销。
2. 不能保证遍历的是最新内容。

## 参考：

https://mp.weixin.qq.com/s/q1r9Pno6ANUzZ9wMzA-JSg

https://blog.csdn.net/xiangyuenacha/article/details/84253630

https://www.jianshu.com/p/bee159e0bd49