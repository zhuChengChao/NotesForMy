# Java：包装类

> 对 Java 中的 **包装类** 这个概念，做一个微不足道的小小小小记

## 基本数据&包装类

**四类八种基本数据类型**：

| 数据类型     | 关键字         | 内存占用 | 取值范围                                   |
| ------------ | -------------- | -------- | ------------------------------------------ |
| 字节型       | byte           | 1个字节  | -128~127                                   |
| 短整型       | short          | 2个字节  | -32768~32767                               |
| 整型         | int（默认）    | 4个字节  | -2^31~2^31 - 1<br />-2147483648~2147483647 |
| 长整型       | long           | 8个字节  | -2^63~2^63 - 1                             |
| 单精度浮点数 | float          | 4个字节  | 1.4013E-45~3.4028E+38                      |
| 双精度浮点数 | double（默认） | 8个字节  | 4.9E-324~1.7977E+308                       |
| 字符型       | char           | 2个字节  | 0-65535                                    |
| 布尔类型     | boolean        | 1个字节  | true，false                                |

**对应的包装类型**：

| 基本类型 | 对应的包装类（位于java.lang包中） |
| -------- | --------------------------------- |
| byte     | Byte                              |
| short    | Short                             |
| int      | Integer                           |
| long     | Long                              |
| float    | Float                             |
| double   | Double                            |
| char     | Character                         |
| boolean  | Boolean                           |

## 装箱和拆箱

自动装箱是 Java 编译器在基本数据类型和对应得包装类之间做的一个转化。比如：把 int 转化成 Integer，double 转化成 Double 等等。反之就是自动拆箱。

原始类型：boolean、char、byte、short、int、long、float、double

封装类型：Boolean、Character、Byte、Short、Integer、Long、Float、Double

### 比较说明-1

```java
Integer a = 1;
Integer b = 2;
Integer c = 3;
Integer d = 3;
Integer e = 321;
Integer f = 321;
Long g = 3L;
Long h = 2L;
int i = 1;

System.out.println(c==d);  		// 有缓存存在，结果为true
System.out.println(e==f);  		// 超出缓存了，地址比较，结果为false
System.out.println(c==(a+b));   // a+b进行了算数运算，触发自动拆箱，因此比较的是数值是否相等，结果为true
System.out.println(i==a);  		// 原始类型和封装类型进行比较时，封装类型会自动拆箱成基本类型后再进行比较,结果为true
System.out.println(c.equals(a+b));  // a+b算数运算，自动拆箱，使用equals时自动装箱为Integer类型的数据，故结果为true
System.out.println(g==(a+b));       // a+b算数运算，自动拆箱，因此比较的是数值是否相等，结果为true
System.out.println(g.equals(a+b));  // a+b算数运算，自动拆箱，使用equals时自动装箱为Integer类型的数据，g为Long类型的数据，类型不一致，因此结果为false
System.out.println(g.equals(a+h));  // 由于此时h为Long类型数据，因此装箱为Long类型数据，比较结果为true
```

### 比较说明-2

```java
Integer x = 100;
Integer y = new Integer(100);
Integer z = new Integer(100);

// 1.Integer ... 和 new Integer:
//   new Integer 会创建对象，存储在堆中
//   而Integer在[-128,127]中，从缓存中取，否则会 new Integer
//   所以Integer和new Integer进行==比较的话，肯定为false;
// Integer和new Integer 进行equals比较的话，肯定为true
System.out.println(x==y);        // false
System.out.println(x.equals(y)); // true
// 2.new Integer和new Integer进行==比较的时候，肯定为false,而进行equals比较的时候，肯定为true
// 原因是new的时候，会在堆中创建对象，分配的地址不同，==比较的是内存地址，所以肯定不同
System.out.println(y==z);         // false
System.out.println(y.equals(z));  // true
```

## Integer vs int

1. int 是 Java 的八种基本数据类型之一，而 Integer 是 Java 为 int 类型提供的封装类；
2. int 型变量的默认值是 0，Integer 变量的默认值是 null，这一点说明 Integer 可以区分出未赋值和值为 0 的区分；
3. Integer 变量必须实例化后才可以使用，而 int 不需要。

**关于 Integer 和 int 的比较的延伸：**

1. 由于 Integer 变量实际上是对一个 Integer 对象的引用，所以两个通过 new 生成的 Integer 变量永远是不相等的，因为其内存地址是不同的；

   ```java
   Integer i = new Integer(100);
   Integer j = new Integer(100);
   System.out.println(i==j);  // false
   ```

2. Integer 变量和 int 变量比较时，只要两个变量的值是相等的，则结果为  true。因为包装类 Integer 和基本数据类型 int 类型进行比较时，**Java 会自动拆包装类为 int**，然后进行比较，实际上就是两个 int 型变量在进行比较；

   ```java
   Integer i = new Integer(100);
   System.out.println(i==100);  // true
   ```

3. 非 new 生成的 Integer 变量和 new Integer() 生成的变量进行比较时，结果为 false。因为**非 new 生成的 Integer 变量指向的是 Java 常量池中的对象**（但是是有范围的，具体见下方），而 new Integer() 生成的变量指向堆中新建的对象，两者在内存中的地址不同；

   ```java
   Integer i = new Integer(100);
   Integer j = 100;
   System.out.println(i==j);  // false
   ```

4. 对于两个非 new 生成的 Integer 对象进行比较时，如果两个变量的值在区间 [-128, 127] 之间，则比较结果为 true，否则为 false。Java 在编译 Integer i = 100 时，会编译成 `Integer i = Integer.valueOf(100)`，而 Integer 类型的 valueOf 的源码如下所示：

   ```java
   Integer i = 127;
   Integer j = 127;
   System.out.println(i==j);  // true
   
   Integer i = 128;
   Integer j = 128;
   System.out.println(i==j);  // false
   
   // valueOf 的源码
   public static Integer valueOf(int i) {
       if (i >= IntegerCache.low && i <= IntegerCache.high)
           return IntegerCache.cache[i + (-IntegerCache.low)];
       return new Integer(i);
   }
   ```

   从上面的代码中可以看出：Java 对于 [-128, 127] 之间的数会进行**缓存**，比如：Integer i = 127，会将 127 进行缓存，下次再写 Integer j = 127 的时候，就会直接从缓存中取出，而对于这个区间之外的数就需要 new 了。

针对上述第4点，引申出**包装类的缓存**：

- Boolean：全部缓存
- Byte：全部缓存
- Character：<= 127 缓存
- Short：-128 ~ 127 缓存
- Long：-128 ~ 127 缓存
- Integer：-128 ~ 127 缓存
- Float：没有缓存
- Doulbe：没有缓存

## 参考

https://blog.csdn.net/qq_35571554/article/details/82876774

https://blog.csdn.net/aoxiangzhe/article/details/81297157

https://www.cnblogs.com/javatech/p/3650460.html