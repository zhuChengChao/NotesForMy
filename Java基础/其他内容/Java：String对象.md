# Java：String对象

> 对 Java 中的 **String** 对象，做一个微不足道的小小小小记

## 字节和字符的区别

### 字节 byte：

1. 一个字节包含8个位(bit)，因此byte的取值范围为-128~127；
2. 字节是存储容量的基本单位；

### 字符 char：

1. 字符是数字、字母、汉字以及其他语言的各种符号；
2. Java 采用 unicode 来表示字符，**Java 中的一个 char 是2个字节**，一个中文或英文字符的 unicode 编码都占2个字节；
3. 在 GB2312 编码或 GBK 编码中，一个英文字母字符存储需要1个字节，一个汉字字符存储需要2个字节；
4. 在 UTF-8 编码中，一个英文字母字符存储需要1个字节，一个汉字字符储存需要3到4个字节；
5. 在 UTF-16 编码中，一个英文字母字符存储需要2个字节，一个汉字字符储存需要3到4个字节（Unicode扩展区的一些汉字存储需要4个字节）；

## String 常用方法

`indexOf()`：返回指定字符的索引

`charAt()`：返回指定索引处的字符

`replace()`：字符串替换

`trim()`：去除字符串两端空白

`split()`：分割字符串，返回一个分割后的字符串数组

`getBytes()`：返回字符串的 byte 类型数组

`length()`：返回字符串长度

`toLowerCase()`：将字符串转成小写字母

`toUpperCase()`：将字符串转成大写字符

`substring()`：截取字符串

`equals()`：字符串比较

> 这些方法还是挺好用的，特别是在刷题的时候:dog2:

## String 为不可变类

### 不可变类

类在被实例化之后，不可用被重新赋值，Java 提供的八个包装类和 String 类都是不可变类。

**创建自定义不可变类需要遵守的规则：**

1. 使用 private 和 final 修饰成员变量。
2. 提供带参构造方法，用于初始化成员变量。
3. 不要为成员变量提供 setter 方法。
4. 如果成员变量中有可变类时需要重写 Object 中的 hashCode 方法和 equals 方法。

### String 部分源码

通过String的源码可以看到

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
	/** Cache the hash code for the string */
    private int hash; // Default to 0
}
```

可以看到 String 的本质是一个 char 数组，是对字符串数组的封装，并且是被 final 修饰的，创建后不可改变。

在 Java 中将 String 设计成不可变的是综合考虑到各种因素的结果。**主要的原因主要有以下4点**：

1. **便于实现字符串池（String pool）**：字符串常量池是 Java 堆内存中一个特殊的存储区域, 当创建一个 String 对象时，假如此字符串值已经存在于常量池中，则不会创建一个新的对象，而是引用已经存在的对象；

   > 即在堆中有一个 StringTable，用于存放字符串常量，具体内容在JVM笔记中说明

2. **加快字符串处理速度：**由于 String 是不可变的，保证了 hashcode 的唯一性，于是在创建对象时其 hashcode就可以放心的缓存了，不需要重新计算。这也就是 Map 喜欢将 String 作为 Key 的原因，处理速度要快过其它的键对象。所以 HashMap 中的键往往都使用 String。

3. **避免安全问题：**String 被许多的 Java 类(库)用来当做参数，例如：网络连接地址 URL、文件路径 path、还有反射机制所需要的 String 参数等， 假如 String 不是固定不变的，将会引起各种安全隐患。

4. **使多线程安全：**

   ```java
   // 下面的例子中可以看到：可变的 StringBuffer 参数就被改变了，而String传入的参数却未被修改
   public class test {
       // 不可变的String
       public static String appendStr(String s) {
           s += "bbb";
           return s;
       }
   
       // 可变的StringBuilder
       public static StringBuilder appendSb(StringBuilder sb) {
           return sb.append("bbb");
       }
   
       public static void main(String[] args) {
           String s = new String("aaa");
           String ns = test.appendStr(s);
           System.out.println("String aaa>>>" + s.toString());  // String aaa>>>aaa
           // StringBuilder做参数
           StringBuilder sb = new StringBuilder("aaa");
           StringBuilder nsb = test.appendSb(sb);
           System.out.println("StringBuilder aaa >>>" + sb.toString());  // StringBuilder aaa >>>aaabbb
       }
   }
   ```

   所以 String 不可变的安全性就体现在这里。在并发场景下，多个线程同时读一个资源，是安全的，不会引发竞争，但对资源进行写操作时是不安全的，不可变对象不能被写，所以保证了多线程的安全。

## String 对象创建的区别

**创建方式1：**`String str = "i"`

**创建方式2：**`String str = new String("i")`

上述两者创建方式是不一样的，因为内存的分配方式不一样。`String str = "i"` 的方式，Java 虚拟机会将其分配到常量池中；而 `String str = new String("i")` 则会被分到堆内存中。

```java
public class StringTest {
    public static void main(String[] args) {
        String str1 = "abc";
        String str2 = "abc";
        String str3 = new String("abc");
        String str4 = new String("abc");

        System.out.println(str1 == str2);      // true
        System.out.println(str1 == str3);      // false
        System.out.println(str3 == str4);      // false
        System.out.println(str3.equals(str4)); // true
    }
}
```

**进一步说明：**

在执行 `String  str1 = "abc"` 的时候，JVM 会首先检查字符串常量池中是否已经存在该字符串对象，如果已经存在，那么就不会再创建了，直接返回该字符串在字符串常量池中的内存地址；如果该字符串还不存在字符串常量池中，那么就会在字符串常量池中创建该字符串对象，然后再返回。所以在执行 `String  str2 = "abc"` 的时候，因为字符串常量池中已经存在 “abc” 字符串对象了，就不会在字符串常量池中再次创建了，所以栈内存中 str1 和 str2 的内存地址都是指向 "abc" 在字符串常量池中的位置，所以 `str1 == str2` 的运行结果为 true。

而在执行 `String  str3 = new  String("abc")` 的时候，JVM 会首先检查字符串常量池中是否已经存在 “abc” 字符串，如果已经存在，则不会在字符串常量池中再创建了；如果不存在，则就会在字符串常量池中创建 "abc" 字符串对象，**然后再到堆内存中再创建一份字符串对象**，把字符串常量池中的 "abc" 字符串内容拷贝到内存中的字符串对象中，然后返回堆内存中该字符串的内存地址，即栈内存中存储的地址是堆内存中对象的内存地址。`String  str4 = new String("abc")` 是在堆内存中又创建了一个对象，所以 `str3 == str4` 运行的结果是 false。

![stringtest](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251534350.PNG)

> 补充：
>
> ```java
> String str5="ab"+"c";
> 
> System.out.println(str1 == str5);  // true
> ```
>
> 此时编译器做了相应的优化，先把字符串拼接，再在常量池中查找这个字符串是否存在，如果存在，则让变量直接引用该字符串，故 str5 也指向字符串常量区中的 'abc'。

## String&StringBuilder&StringBuffer

1. **执行效率：**StringBuilder > StringBuffer > String

   **String最慢的原因：**String 为字符串常量，而 StringBuilder 和 StringBuffer 均为字符串变量，即 String 对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的。

2. **线程安全：**在线程安全上，StringBuilder 是线程不安全的，而 StringBuffer 是线程安全的

   如果一个 StringBuffer 对象在字符串缓冲区被多个线程使用时，StringBuffer 中很多方法可以带有**synchronized关键字**，所以可以保证线程是安全的，但 StringBuilder 的方法则没有该关键字，所以不能保证线程安全，有可能会出现一些错误的操作。所以如果要进行的操作是多线程的，那么就要使用StringBuffer，但是在单线程的情况下，还是建议使用速度比较快的 StringBuilder。

   > **StringBuffer 的补充：**
   >
   > 说明：StringBuffer 中并不是所有方法都使用了 Synchronized 修饰来实现同步：
   >
   > ```java
   > // 带关键字synchronized
   > @Override
   > public synchronized StringBuffer insert(int offset, Object obj) {
   >     toStringCache = null;
   >     super.insert(offset, String.valueOf(obj));
   >     return this;
   > }
   > 
   > // 不带关键字synchronized
   > @Override
   > public StringBuffer insert(int dstOffset, CharSequence s) {
   >  // Note, synchronization achieved via invocations of other StringBuffer methods
   >  // after narrowing of s to specific type
   >  // Ditto for toStringCache clearing
   >  super.insert(dstOffset, s);
   >  return this;
   > }
   > ```

3. 总结一下：

   **String：适用于少量的字符串操作的情况**

   **StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况**

   **StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况**

## 参考

https://www.cnblogs.com/wugongzi/p/11334052.html

https://www.php.cn/java/base/437469.html

https://www.cnblogs.com/wkfvawl/p/11693260.html

https://www.cnblogs.com/su-feng/p/6659064.html

https://www.cnblogs.com/nov5026/p/7248509.html

https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

