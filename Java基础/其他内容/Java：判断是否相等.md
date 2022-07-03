# Java：判断是否相等

> 对 Java 中的判断是否相等，即判断两数/两对象是否相等，做一个微不足道的小小小小结

## `==` 判断

对于基本类型和引用类型 == 的效果是不同的，如下：

* 基本类型：比较的**值是否相同**；
* 引用类型：比较的是是否引用同一对象，**即引用对象的地址值是否相等**

示例：

```java
int int_a = 10;
int int_b = 10;
System.out.println(int_a==int_b);  // true

String stra = "abc";
String strb = "abc";
String strc = new String("abc");  // 通过new String()方法重新开辟了内存空间
String strd = new String("abc");
System.out.println(stra==strb);  // true
System.out.println(stra==strc);  // false
System.out.println(strc==strd);  // false
```

## `equals` 方法

**`equals()` 方法：首先，原始的 equals 方法，只是用于比较两个对象的内存地址是否为同一个，如下：**

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

而如果没有对 equals 方法进行重写，则比较的是引用类型的变量所指向的对象的地址（很多类重写了 equals 方法，比如 String、Integer 等把它变成了值比较，所以一般情况下 equals 比较的是值是否相等）。

如下代码所示：

```java
// Person没有对equals方法进行重新，而String对equals方法进行了重写
private class Person{
    public String name;

    public Person(String name) {
        this.name = name;
    }
}

@Test
public void test02(){
    String strc = new String("abc");
    String strd = new String("abc");
    System.out.println(strc.equals(strd));  // true

    Person p1 = new Person("张三");
    Person p2 = new Person("张三");
    System.out.println(p1.equals(p2));  // false
}
```

查看 Person 类中的 equals 方法：本质上 equals 就是 ==，即比较两个对象地址

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

查看 String 类中的 equals 方法：

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;  // 地址相同直接返回true
    }
    if (anObject instanceof String) {  // 判断是否为String对象
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {  // 长度相同才进行比较
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {			// 按字符比较
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

> 同时需要注意的是：equals 不能用于基础类型的比较，因为 equals 是一个方法，而基础类似并不是一个对象，没有对应的方法

## `hashCode()` 方法

### `hashCode()` 方法：

提到 hashCode，自然联想到哈希表，通过 hashCode 方法，将 key 映射到哈希表中的一个位置，从而达到最好情况下以 O(1) 的时间复杂度来查询。

```java
// Object类的源码中，hashcode() 是一个 native 方法，哈希值的计算利用的是内存的地址
public native int hashCode();
```

### 问：为什么重写 `equals()` 就一定要重写 `hashCode()` 方法？

**`equals()` 方法：**

首先，原始的 equals 方法，只是用于比较两个对象的内存地址是否为同一个，如下：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

而重写 `equals()` 方法，是**为了实现当两个对象指向的内存地址相同或者两个对象各自字段值相同，那么标识其为同一个对象**。

**为什么要重写？**

这个问题应该是有个前提，就是你需要用到 HashMap、HashSet 等 Java 集合，用不到哈希表的话，其实仅仅重写 `equals()` 方法也可以。而工作中的场景是常常用到 Java 集合，所以 Java 官方建议重写 `equals()` 就一定要重写 `hashCode()` 方法。 

1. 提高效率：对于对象集合的判重，如果一个集合含有 10000 个对象实例，仅仅使用 `equals()` 方法的话，那么对于一个对象判重就需要比较 10000 次，随着集合规模的增大，时间开销是很大的。但是同时使用哈希表的话，就能快速定位到对象的大概存储位置，并且在定位到大概存储位置后，后续比较过程中，**如果两个对象的 hashCode 不相同，也不再需要调用 equals() 方法，从而大大减少了 equals() 比较次数**。 
2. 在实际应用过程中，如果仅仅重写了 `equals()`，而没有重写 `hashCode()` 方法，会出现什么情况？**字段属性值完全相同的两个对象因为 hashCode 不同，所以在 HashMap 中的 table 数组的下标不同，从而这两个对象就会同时存在于集合中，所以重写 `equals()` 就一定要重写 `hashCode()` 方法。**

### `hashCode()` 与 `equals()` 的相关规定：

1. 如果两个对象相等，则 hashCode 一定也是相同的；
2. 两个对象相等，对两个对象分别调用 equals 方法都返回 true；
3. 两个对象有相同的 hashCode 值，它们也不一定是相等的；
4. equals 方法被覆盖过，则 hashCode 方法也必须被覆盖；
5. `hashCode()` 的默认行为是对堆上的对象产生独特值。如果没有重写 `hashCode()`，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。

对上述第3点进行说明：**两个对象的 `hashCode()` 相同，则 `equals()` 不一定为 true**，如下

```java
@Test
public void test03(){
    String str1 = "通话";
    String str2 = "重地";

    System.out.println("str1 hashCode:"+str1.hashCode());  // 1179395
    System.out.println("str2 hashCode:"+str2.hashCode());  // 1179395
}
```

在散列表中，`hashCode()`相等，只是说明两个键值对的哈希值相等，但是哈希值相等，并不一定可以得出键值对相等。

## 参考

https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

https://blog.csdn.net/qq_41701956/article/details/103223461

https://blog.csdn.net/sixingmiyi39473/article/details/78306296

